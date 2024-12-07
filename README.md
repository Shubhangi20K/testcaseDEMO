package com.epay.transaction.service;

/*
  Class Name:CustomerService
  *
  Description:
  *
  Author:V1014352(Ranjan Kumar)
  <p>
  Copyright (c) 2024 [State Bank of India]
  All right reserved
  *
  Version:1.0
 */

import com.epay.transaction.dao.CustomerDao;
import com.epay.transaction.dto.CustomerDto;
import com.epay.transaction.dto.MerchantDto;
import com.epay.transaction.exceptions.TransactionException;
import com.epay.transaction.model.request.CustomerRequest;
import com.epay.transaction.model.response.CustomerResponse;
import com.epay.transaction.model.response.TransactionResponse;
import com.epay.transaction.util.EPayIdentityUtil;
import com.epay.transaction.util.EncryptionDecryptionUtil;
import com.epay.transaction.util.ErrorConstants;
import com.epay.transaction.util.UniqueIDGenerator;
import com.epay.transaction.util.enums.Status;
import com.epay.transaction.validator.CustomerValidator;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.sbi.epay.authentication.model.EPayPrincipal;
import com.sbi.epay.logging.utility.LoggerFactoryUtility;
import com.sbi.epay.logging.utility.LoggerUtility;
import lombok.RequiredArgsConstructor;
import org.joda.time.DateTimeUtils;
import org.springframework.stereotype.Service;

import java.text.MessageFormat;
import java.util.List;

@Service
@RequiredArgsConstructor
public class CustomerService {
    private final CustomerDao customerDao;
    private final CustomerValidator customerValidator;
    private final UniqueIDGenerator uniqueIDGenerator;
    private final EncryptionDecryptionUtil encryptionDecryptionUtil;
    private final ObjectMapper objectMapper;

    LoggerUtility logger = LoggerFactoryUtility.getLogger(CustomerService.class);

    public TransactionResponse<CustomerResponse> saveCustomer(CustomerRequest customerRequest) {
        EPayPrincipal ePayPrincipal = EPayIdentityUtil.getUserPrincipal();
        //Step 1 : Get Merchant for finding the Keys
        logger.info("get Merchant Id");
        MerchantDto merchantDto = customerDao.getActiveMerchantByMID(ePayPrincipal.getMid());
        //Step 2 : Decrypt the customerRequest
        CustomerDto customerDto = buildCustomerDTO(customerRequest.getCustomerRequest(), merchantDto);
        //Step 3 : Validated customerRequest
        customerValidator.validateCustomerRequest(customerDto);
        //Step 4 : Generate CustomerId
        customerDto.setCustomerId(uniqueIDGenerator.generateUniqueCustomerId());
        customerDto.setCreatedDate(DateTimeUtils.currentTimeMillis());
        customerDto.setUpdatedDate(DateTimeUtils.currentTimeMillis());
        customerDto = customerDao.saveCustomer(customerDto);
        //Step 5 : Build Customer Response and send to controller
        String customerData = buildCustomerData(customerDto, merchantDto);
        CustomerResponse customerResponse = CustomerResponse.builder().customerId(customerDto.getCustomerId()).customerResponse(customerData).build();
        return TransactionResponse.<CustomerResponse>builder().data(List.of(customerResponse)).status(1).count(1L).build();
    }

    public TransactionResponse<CustomerResponse> getCustomerByCustomerId(String customerId) {
        CustomerDto customerDto = customerDao.getCustomerByCustomerId(customerId).orElseThrow(() -> new TransactionException(ErrorConstants.NOT_FOUND_ERROR_CODE, MessageFormat.format(ErrorConstants.NOT_FOUND_ERROR_MESSAGE, "Valid Customer")));
        MerchantDto merchantDto = customerDao.getActiveMerchantByMID(customerDto.getMId());
        String customerData = buildCustomerData(customerDto, merchantDto);
        CustomerResponse customerResponse = CustomerResponse.builder().customerId(customerDto.getCustomerId()).customerResponse(customerData).build();
        return TransactionResponse.<CustomerResponse>builder().data(List.of(customerResponse)).status(1).count(1L).build();
    }

    /**
     * Service method to update customer status
     * @param customerId
     * @param status
     * @return CustomerResponse
     */
    public TransactionResponse<CustomerResponse> updateCustomerStatus(String customerId, String status) {
        //Step 1 : Get Customer by mid
        CustomerDto customerDto = customerDao.getCustomerByCustomerId(customerId).orElseThrow(() ->
                new TransactionException(ErrorConstants.NOT_FOUND_ERROR_CODE, MessageFormat.format(ErrorConstants.NOT_FOUND_ERROR_MESSAGE, "Valid Customer")));//Step 2 : Decrypt the customerRequest
        MerchantDto merchantDto = customerDao.getActiveMerchantByMID(customerDto.getMId());
        //Status validation and set the status to customer DTO
        customerDto.setStatus(Status.getStatus(status).name());
        //Call DAO method
        customerDao.updateCustomerStatus(customerDto);
        String customerData = buildCustomerData(customerDto, merchantDto);
        CustomerResponse customerResponse = CustomerResponse.builder().customerId(customerDto.getCustomerId()).customerResponse(customerData).build();
        return TransactionResponse.<CustomerResponse>builder().data(List.of(customerResponse)).status(1).count(1L).build();
    }

    private CustomerDto buildCustomerDTO(String customerRequest, MerchantDto merchantDto) {
        try {
            String decryptedCustomerRequest = encryptionDecryptionUtil.decryptRequest(customerRequest, merchantDto);
            return objectMapper.readValue(decryptedCustomerRequest, CustomerDto.class);
        } catch (JsonProcessingException e) {
            logger.error("Error in buildCustomerDTO  ", e);
            throw new TransactionException(ErrorConstants.INVALID_ERROR_CODE, MessageFormat.format(ErrorConstants.INVALID_ERROR_MESSAGE, customerRequest, "Request is not proper"));
        }
    }

    private String buildCustomerData(CustomerDto customerDto, MerchantDto merchantDto) {
        try {
            return encryptionDecryptionUtil.encryptRequest(objectMapper.writeValueAsString(customerDto), merchantDto);
        } catch (JsonProcessingException e) {
            logger.error("Error in buildCustomerData  ", e);
            throw new TransactionException(ErrorConstants.INVALID_ERROR_CODE, MessageFormat.format(ErrorConstants.INVALID_ERROR_MESSAGE, customerDto, "Request is not proper"));
        }
    }
}

