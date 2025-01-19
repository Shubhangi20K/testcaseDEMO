@GetMapping(path = "/deviceDetail/{orderHash}")
@Operation(summary = "Get Device Details by OrderHash")
public DeviceDetails getDeviceDetailsByPath(@PathVariable String orderHash) 
        throws InstantiationException, IllegalAccessException {

    logger.info("Fetching Device Details with orderHash {}", orderHash);

    return tokenService.getDeviceDetails(orderHash);
}

@Repository
public interface CaptureDetailInfoRepository extends JpaRepository<DeviceDetails, Long> {

    Optional<DeviceDetails> findByOrderHash(String orderHash);
}
@Service
public class TokenService {

    @Autowired
    private CaptureDetailInfoRepository captureDetailInfoRepository;

    public DeviceDetails getDeviceDetails(String orderHash) throws InstantiationException, IllegalAccessException {
        logger.debug("Fetching device details for orderHash {}", orderHash);

        // Fetch data from the repository
        return captureDetailInfoRepository.findByOrderHash(orderHash)
                .orElseThrow(() -> new IllegalAccessException("Device details not found for orderHash: " + orderHash));
    }
}
@GetMapping(path = "/deviceDetail")
@Operation(summary = "Get Device Details by OrderHash")
public DeviceDetails getDeviceDetails(@RequestParam String orderHash) 
        throws InstantiationException, IllegalAccessException {

    logger.info("Fetching Device Details with orderHash {}", orderHash);

    // Retrieve device details from the service layer
    return tokenService.getDeviceDetails(orderHash);
}
