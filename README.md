package com.epay.transaction.externalservice;

import com.epay.transaction.client.ApiClient;
import com.epay.transaction.exceptions.TransactionException;
import com.epay.transaction.model.request.DcmsRequest;
import com.epay.transaction.model.request.EISPayload;
import com.epay.transaction.model.request.EisCardNumberRequest;
import com.epay.transaction.model.response.ChannelDetailsResponse;
import com.epay.transaction.model.response.EisResponse;
import com.epay.transaction.model.response.TransactionResponse;
import com.epay.transaction.util.DcmsConstants;
import com.epay.transaction.util.EisCrypto;
import com.epay.transaction.util.ErrorConstants;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.sbi.epay.logging.utility.LoggerFactoryUtility;
import com.sbi.epay.logging.utility.LoggerUtility;
import org.apache.commons.lang3.ObjectUtils;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.MediaType;
import org.springframework.stereotype.Service;
import java.net.URI;
import java.text.MessageFormat;
import java.util.List;

/**
 *
 *  Copyright (c) [2024] [State Bank of India]
 *  All rights reserved.
 *
 *  Author:@V0000001(Shubhangi Kurelay)
 *  Version:1.0
 *
 */

@Service
public class TransactionServicesClient extends ApiClient {

    private static final LoggerUtility logger = LoggerFactoryUtility.getLogger(TransactionServicesClient.class);
    private ObjectMapper objectMapper;

    @Value("${external.api.transaction.services.base.path}")
    private String dcmsExternalAPIUrl;

    public TransactionServicesClient(String baseUrl) {
        super(baseUrl);

    }

    /**
     * Get ECOM Flag in response object
     * @param cardNumberRequest

     * @return
     * @throws JsonProcessingException
     */
    public TransactionResponse<String> getEisDCMSCardNumberValidation(EisCardNumberRequest cardNumberRequest) throws JsonProcessingException {
        logger.info("Get Eis Response ");
        EisCrypto eisCrypto = new EisCrypto();
        String key=eisCrypto.getSecureRandomKey(128);
        logger.info("key : {}",key);

        String dcmsRequest = DcmsRequest.builder()
                .requestReferenceNumber(eisCrypto.generateReqRefNo(DcmsConstants.SOURCE_ID, DcmsConstants.CLIENT_Id))
                .sourceId(DcmsConstants.SOURCE_ID)
                .destination(DcmsConstants.DESTINATION)
                .txnType(DcmsConstants.TXT_TYPE)
                .txnSubType(DcmsConstants.TXN_SUB_TYPE)
                .eispayload(EISPayload.builder().cardNumber(cardNumberRequest.getCardNumber()).actionType(DcmsConstants.ACTION_TYPE).bankCode(DcmsConstants.CLIENT_Id).sourceId(DcmsConstants.EIS_SOURCE_ID).build())
                .toString();
        logger.info("Request object : {}", dcmsRequest);

        String encryptedRequest = eisCrypto.encrypt(dcmsRequest, key);
        logger.info("Encrypted request object : {}",encryptedRequest);

        URI url = URI.create(dcmsExternalAPIUrl);
        String eisResponse = getWebClient()
                .post()
                .uri(url)
                .contentType(MediaType.APPLICATION_JSON)
                .bodyValue(encryptedRequest)
                .retrieve()
                .bodyToMono(String.class)  // Deserialize the response to GstResponse
                .block();
        logger.info("EIS Response received: {}",eisResponse);

        String ecomFlag = "";
        if (ObjectUtils.isNotEmpty(eisResponse)) {
            String decryptedResult = EisCrypto.decrypt(eisResponse, key);
            EisResponse decreptedEISResponse = objectMapper.readValue(decryptedResult, EisResponse.class);
            logger.info("EIS decrypted response: {}",decreptedEISResponse);
            ecomFlag = getECOMFromResponse(decreptedEISResponse.getChannelDetails());
            if (!ObjectUtils.isNotEmpty(ecomFlag)){
                throw new TransactionException(ErrorConstants.NOT_FOUND_ERROR_CODE, MessageFormat.format(ErrorConstants.NOT_FOUND_ERROR_MESSAGE,"ECOM Flag"));
            }
        }

        logger.info("EIS ecomflag: {}", ecomFlag);
        return TransactionResponse.<String>builder().data(List.of(ecomFlag)).status(1).build();
    }

    /**
     * Get ECOM flag to the response
     * @param channelDetailsResponseList
     * @return String ECOM flag
     */
    private String getECOMFromResponse(List<ChannelDetailsResponse> channelDetailsResponseList){
        logger.info("Get ECOM Response flag");
        ChannelDetailsResponse flag = new ChannelDetailsResponse();
        channelDetailsResponseList.forEach(chanel ->{
                    if ("ECOM".equals(chanel.getCId())) {
                        flag.setFlag(chanel.getFlag());
                    }
        });
        return flag.getFlag();
    }


}

