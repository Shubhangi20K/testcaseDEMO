import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class EncryptionController {

    private static final String ENCRYPTION_KEY = "1234567890123456"; // 16-byte key for AES

    @GetMapping("/encrypt")
    public String encrypt(@RequestParam String plaintext) {
        try {
            return EncryptionUtils.encrypt(plaintext, ENCRYPTION_KEY);
        } catch (Exception e) {
            return "Error encrypting the text: " + e.getMessage();
        }
    }

    @GetMapping("/decrypt")
    public String decrypt(@RequestParam String ciphertext) {
        try {
            return EncryptionUtils.decrypt(ciphertext, ENCRYPTION_KEY);
        } catch (Exception e) {
            return "Error decrypting the text: " + e.getMessage();
        }
    }
}
