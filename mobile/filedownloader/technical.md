# Technical Documentation

## Download File using Retrofit
```java
class FileDownloadScreenViewModel : ViewModel() {
    private val api: FileDownloadApi = createRetrofitApi()
    var state by mutableStateOf<FileDownloadScreenState>(FileDownloadScreenState.Idle)
        private set

    fun downloadFile() {
        viewModelScope.launch(Dispatchers.IO) {
            val timestamp = System.currentTimeMillis()
            api.downloadZipFile()
                .saveFile(timestamp.toString())
                .collect { downloadState ->
                    state = when (downloadState) {
                        is DownloadState.Downloading -> {
                            FileDownloadScreenState.Downloading(progress = downloadState.progress)
                        }
                        is DownloadState.Failed -> {
                            FileDownloadScreenState.Failed(error = downloadState.error)
                        }
                        DownloadState.Finished -> {
                            FileDownloadScreenState.Downloaded
                        }
                    }
                }
        }
    }

    fun onIdleRequested() {
        state = FileDownloadScreenState.Idle
    }

    private sealed class DownloadState {
        data class Downloading(val progress: Int) : DownloadState()
        object Finished : DownloadState()
        data class Failed(val error: Throwable? = null) : DownloadState()
    }

    /*private fun ResponseBody.saveFileSimple(filePostfix: String) {
        val downloadFolder = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DOWNLOADS)
        val file = File(downloadFolder.absolutePath, "file_${filePostfix}.zip")
        byteStream().use { inputStream ->
            file.outputStream().use { outputStream ->
                inputStream.copyTo(outputStream)
            }
        }
    }*/

    private fun ResponseBody.saveFile(filePostfix: String): Flow<DownloadState> {
        return flow {
            emit(DownloadState.Downloading(0))
            val downloadFolder = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DOWNLOADS)
            val destinationFile = File(downloadFolder.absolutePath, "file_${filePostfix}.zip")

            try {
                byteStream().use { inputStream ->
                    destinationFile.outputStream().use { outputStream ->
                        val totalBytes = contentLength()
                        val buffer = ByteArray(DEFAULT_BUFFER_SIZE)
                        var progressBytes = 0L

                        var bytes = inputStream.read(buffer)
                        while (bytes >= 0) {
                            outputStream.write(buffer, 0, bytes)
                            progressBytes += bytes
                            bytes = inputStream.read(buffer)
                            emit(DownloadState.Downloading(((progressBytes * 100) / totalBytes).toInt()))
                        }
                    }
                }
                emit(DownloadState.Finished)
            } catch (e: Exception) {
                emit(DownloadState.Failed(e))
            }
        }
            .flowOn(Dispatchers.IO)
            .distinctUntilChanged()
    }
}
```

## Download multiple files using coroutine
```java
suspend fun downloadLinks(pendingFiles: List<PendingFile>) = coroutineScope {

    val deferredList = pendingFiles.map {
            async(Dispatchers.IO) {
                // runs in parallel in background thread
                try {
                   networkCallToGetData(it)
                } catch (e: Exception) { // might wanna adjust this depending on your use case
                   null // null here means failure, alternately you could use a sealed class with success and failure
                }
            }

           // Waiting all requests are finished without blocking the current thread
            val listOfReturnData = deferredList.awaitAll()

            val (success, failed) = listOfReturnData.partition { 
                 it != null
            }

           TODO() // rest of your code
    }
}
```

### Pause and Resume download
```java
object FileDownloader{

    private val Service by lazy { serviceBuilder().create<FileDownloaderInterface>(FileDownloaderInterface::class.java) }

    val baseUrl = "http://www.your-website-base-url.com"

    private fun serviceBuilder(): Retrofit {
        //--- OkHttp client ---//
        val okHttpClient = OkHttpClient.Builder()
                .readTimeout(60, TimeUnit.SECONDS)
                .connectTimeout(60, TimeUnit.SECONDS)

        //--- Add authentication headers ---//
        okHttpClient.addInterceptor { chain ->
            val original = chain.request()

            // Just some example headers
            val requestBuilder = original.newBuilder()
                    .addHeader("Connection","keep-alive")
                    .header("User-Agent", "downloader")

            val request = requestBuilder.build()
            chain.proceed(request)
        }

        //--- Add logging ---//

        if (BuildConfig.DEBUG) {
            // development build
            val logging = HttpLoggingInterceptor()
            logging.setLevel(HttpLoggingInterceptor.Level.BASIC)
            // NOTE: do NOT use request BODY logging or it will not work!

            okHttpClient.addInterceptor(logging)
        }


        //--- Return Retrofit class ---//
        return Retrofit.Builder()
                .client(okHttpClient.build())
                .baseUrl(baseUrl)
                .build()
    }

    suspend fun downloadOrResume(
            url:String, destination: File,
            headers:HashMap<String,String> = HashMap<String,String>(),
            onProgress: ((percent: Int, downloaded: Long, total: Long) -> Unit)? = null
        ){

        var startingFrom = 0L
        if(destination.exists() && destination.length()>0L){
            startingFrom = destination.length()
            headers.put("Range","bytes=${startingFrom}-")
        }
        println("Download starting from $startingFrom - headers: $headers")

        download(url,destination,headers,onProgress)
    }

    suspend fun download(
            url:String,
            destination: File,
            headers:HashMap<String,String> = HashMap<String,String>(),
            onProgress: ((percent: Int, downloaded: Long, total: Long) -> Unit)? = null
        ) {
        println("---------- downloadFileByUrl: getting response -------------")
        val response = Service.downloadFile(url,headers).awaitResponse()
        handleDownloadResponse(response,destination,onProgress)
    }

    fun handleDownloadResponse(
            response:Response<ResponseBody>,
            destination:File,
            onProgress: ((percent: Int, downloaded: Long, total: Long) -> Unit)?
    ) {
        println("-- downloadFileByUrl: parsing response! $response")


        var startingByte = 0L
        var endingByte = 0L
        var totalBytes = 0L


        if(!response.isSuccessful) {
            throw HttpException(response)
            //java.lang.IllegalStateException: Error downloading file: 416, Requested Range Not Satisfiable; Response Response{protocol=http/1.1, code=416, message=Requested Range Not Satisfiable, u
        }
        val contentLength = response.body()!!.contentLength()

        if (response.code() == 206) {
            println("- http 206: Continue download")
            val matcher = Pattern.compile("bytes ([0-9]*)-([0-9]*)/([0-9]*)").matcher(response.headers().get("Content-Range"))
            if (matcher.find()) {
                startingByte = matcher.group(1).toLong()
                endingByte = matcher.group(2).toLong()
                totalBytes = matcher.group(3).toLong()
            }
            println("Getting range from $startingByte to ${endingByte} of ${totalBytes} bytes" )
        } else {
            println("- new download")
            endingByte = contentLength
            totalBytes = contentLength
            if (destination.exists()) {
                println("Delete previous download!")
                destination.delete()
            }
        }


        println("Getting range from $startingByte to ${endingByte} of ${totalBytes} bytes" )
        val sink: BufferedSink
        if (startingByte > 0) {
            sink = Okio.buffer(Okio.appendingSink(destination))
        } else {
            sink = Okio.buffer(Okio.sink(destination))
        }


        var lastPercentage=-1
        var totalRead=startingByte
        sink.use {
            it.writeAll(object : ForwardingSource(response.body()!!.source()) {

                override fun read(sink: Buffer, byteCount: Long): Long {
                    //println("- Reading... $byteCount")
                    val bytesRead = super.read(sink, byteCount)

                    totalRead += bytesRead

                    val currentPercentage = (totalRead * 100 / totalBytes).toInt()
                    //println("Progress: $currentPercentage - $totalRead")
                    if (currentPercentage > lastPercentage) {
                        lastPercentage = currentPercentage
                        if(onProgress!=null){
                            onProgress(currentPercentage,totalRead,totalBytes)
                        }
                    }
                    return bytesRead
                }
            })
        }

        println("--- Download complete!")
    }

    internal interface FileDownloaderInterface{
        @Streaming
        @GET
        fun downloadFile(
                @Url fileUrl: String,
                @HeaderMap headers:Map<String,String>
        ): Call<ResponseBody>
    }
}

// Example usage
val url = "https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-9.4.0-amd64-xfce-CD-1.iso"
val destination = File(context.filesDir, "debian-9.4.0-amd64-xfce-CD-1.iso")

//Optional: you can also add custom headers
val headers = HashMap<String,String>()

try {
    // Start or continue a download, catch download exceptions
    FileDownloader.downloadOrResume(
            url,
            destination,
            headers,
            onProgress = { progress, read, total ->
                println(">>> Download $progress% ($read/$total b)")
            });
}catch(e: SocketTimeoutException){
    println("Download socket TIMEOUT exception: $e")
}catch(e: SocketException){
    println("Download socket exception: $e")
}catch(e: HttpException){
    println("Download HTTP exception: $e")
}
```