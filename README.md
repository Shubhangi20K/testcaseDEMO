@GetMapping(path = "/deviceDetail")
@Operation(summary = "Get Device Details by OrderHash")
public DeviceDetails getDeviceDetails(@RequestParam String orderHash) 
        throws InstantiationException, IllegalAccessException {

    logger.info("Fetching Device Details with orderHash {}", orderHash);

    // Pass the orderHash to the service layer to get the device details
    return tokenService.getDeviceDetails(orderHash);
}
