@RequiredArgsConstructor
@Component
public class EncryptionDecryptionUtil {
    private static final LoggerUtility logger = LoggerFactoryUtility.getLogger(EncryptionDecryptionUtil.class);
    private final EncryptionService encryptionService;
    private final DecryptionService decryptionService;

    public String decryptRequest(String request, MerchantDto merchantDto) {
        SecretKey secretKey = KeyProviderService.getDecryptedMEK(merchantDto.getMek(), merchantDto.getKek(), merchantDto.getAek(), EncryptionDecryptionAlgo.AES_GCM_NO_PADDING, GCMIvLength.MAXIMUM, GCMTagLength.STANDARD);
        return decryptionService.decryptValue(Base64.getDecoder().decode(request), secretKey, EncryptionDecryptionAlgo.AES_GCM_NO_PADDING, GCMIvLength.MAXIMUM, GCMTagLength.STANDARD);
    }

    public String encryptRequest(String request, MerchantDto merchantDto) {
        SecretKey secretKey = KeyProviderService.getDecryptedMEK(merchantDto.getMek(), merchantDto.getKek(), merchantDto.getAek(), EncryptionDecryptionAlgo.AES_GCM_NO_PADDING, GCMIvLength.MAXIMUM, GCMTagLength.STANDARD);
        byte[] encryptedValue = encryptionService.encryptValue(secretKey, request, EncryptionDecryptionAlgo.AES_GCM_NO_PADDING, GCMIvLength.MAXIMUM, GCMTagLength.STANDARD);
        return Base64.getEncoder().encodeToString(encryptedValue);
    }

    public String hashValue(String... values){
        String value = String.join("", values);
        byte[] bytes = HashingService.generateHash(Base64.getDecoder().decode(value), HashAlgorithm.SHA_512);
        return Base64.getEncoder().encodeToString(bytes).replaceAll("[-+^=!/]*", "");
    }

}


import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;

import javax.crypto.SecretKey;
import java.util.Base64;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

class EncryptionServiceTest {

    @InjectMocks
    private YourServiceClass service; // Replace with your class name

    @Mock
    private KeyProviderService keyProviderService;

    @Mock
    private EncryptionService encryptionService;

    @Mock
    private DecryptionService decryptionService;

    @Mock
    private HashingService hashingService;

    @Mock
    private SecretKey secretKey;

    private MerchantDto merchantDto;

    @BeforeEach
    void setUp() {
        MockitoAnnotations.openMocks(this);
        merchantDto = new MerchantDto();
        merchantDto.setMek("mek");
        merchantDto.setKek("kek");
        merchantDto.setAek("aek");
    }

    @Test
    void testDecryptRequest() {
        String request = "encryptedRequest";
        byte[] decodedRequest = Base64.getDecoder().decode(request);
        String decryptedValue = "decryptedValue";

        when(keyProviderService.getDecryptedMEK(
                merchantDto.getMek(), merchantDto.getKek(), merchantDto.getAek(),
 

