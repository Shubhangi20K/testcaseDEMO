    @PostMapping(path = "/deviceDeatil")
    @Operation(summary = "Transaction Token Generation")
    public DeviceDetails generateTransactionToken(@RequestBody DeviceDetailsRequest payLoad) throws InstantiationException, IllegalAccessException {

        logger.info("Transaction Token Request with orderHash {}");
        return tokenService.generateTransactionToken( payLoad);
    }


  public DeviceDetails generateTransactionToken(DeviceDetailsRequest payLoad) throws InstantiationException, IllegalAccessException {
        logger.debug(" Saving Capture Detail Information");
        DeviceDetailsDto deviceDetailsDto = DeviceDetailsDto.builder()
                //  .orderHash(String.valueOf(payLoad.getOrderHash()))
                .clientIp(String.valueOf(payLoad.getClientIp()))
                .createdDate(DateTimeUtils.currentTimeMillis())
                .deviceDetail(Arrays.toString(String.valueOf(payLoad.getDeviceDetail()).getBytes())).build();
        //Save Device information
        return   tokenDao.saveCaptureDeviceInfo(deviceDetailsDto);


    }

       public DeviceDetails saveCaptureDeviceInfo(DeviceDetailsDto deviceDetailsDto) throws InstantiationException, IllegalAccessException {
        DeviceDetails deviceDetails = new DeviceDetails();//objectMapper.convertValue(captureDetailInfoDto,CaptureDeviceInfo.class);
       // deviceDetails.setOrderHash(deviceDetailsDto.getOrderHash());
        deviceDetails.setClientIp(deviceDetailsDto.getClientIp());
        deviceDetails.setDeviceDetails(deviceDetailsDto.getDeviceDetail().getBytes());
        deviceDetails.setCreatedDate(DateTimeUtils.currentTimeMillis());
        return capturedetailinforepository.save(deviceDetails);
      
    }

  <S extends T> S save(S entity);
