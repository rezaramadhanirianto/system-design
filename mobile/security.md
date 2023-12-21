# Security

## E2E Security
End-to-End (E2E) encryption is a security measure that ensures that data is encrypted and can only be decrypted by the intended recipient. In the context of mobile communication, especially on Android devices, E2E encryption is often used to protect the privacy and security of user messages and calls.

Besides messages and calls we can encrypt payment using this method I think.

```kotlin
import android.util.Base64
import javax.crypto.Cipher
import javax.crypto.spec.SecretKeySpec

object EncryptionUtils {

    private const val ALGORITHM = "AES"
    private const val TRANSFORMATION = "AES/ECB/PKCS5Padding"
    private const val SECRET_KEY = "ThisIsASecretKey"

    fun encrypt(text: String): String {
        val cipher = Cipher.getInstance(TRANSFORMATION)
        cipher.init(Cipher.ENCRYPT_MODE, SecretKeySpec(SECRET_KEY.toByteArray(), ALGORITHM))
        val encryptedBytes = cipher.doFinal(text.toByteArray())
        return Base64.encodeToString(encryptedBytes, Base64.DEFAULT)
    }

    fun decrypt(encryptedText: String): String {
        val cipher = Cipher.getInstance(TRANSFORMATION)
        cipher.init(Cipher.DECRYPT_MODE, SecretKeySpec(SECRET_KEY.toByteArray(), ALGORITHM))
        val decryptedBytes = cipher.doFinal(Base64.decode(encryptedText, Base64.DEFAULT))
        return String(decryptedBytes)
    }
}

```

## Local database encryption
Popular local database encryption using sqlcipher.

## References
- <a href="https://developer.mastercard.com/platform/documentation/security-and-authentication/securing-sensitive-data-using-payload-encryption/">How Mastercard secure their payload request</a>
- <a href="https://about.fb.com/news/2023/12/default-end-to-end-encryption-on-messenger/#:~:text=The%20extra%20layer%20of%20security,they%20reach%20the%20receiver's%20device.">How Messenger secure their chat</a>
