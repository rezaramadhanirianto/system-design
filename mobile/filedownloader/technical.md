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