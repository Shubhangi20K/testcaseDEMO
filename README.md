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
