package com.example.demo.service;

import com.example.demo.model.LimitFlagRequest;
import com.example.demo.model.LimitFlagResponse;
import com.example.demo.model.ChannelDetail;
import org.springframework.stereotype.Service;

import java.util.ArrayList;
import java.util.List;

@Service
public class LimitFlagService {

    public LimitFlagResponse getLimitFlag(LimitFlagRequest request) {
        // Create response object
        LimitFlagResponse response = new LimitFlagResponse();
        List<ChannelDetail> channelDetails = new ArrayList<>();

        // Mock data based on the request
        // In a real-world scenario, this logic would involve more complex processing
        channelDetails.add(createChannelDetail("Y", "NA", "DOM", "NA"));
        channelDetails.add(createChannelDetail("N", "NA", "INT", "NA"));
        channelDetails.add(createChannelDetail("Y", "40000", "ATM", "000000000000"));
        channelDetails.add(createChannelDetail("Y", "75000", "POS", "000000000000"));
        channelDetails.add(createChannelDetail("N", "NA", "ECOM", "NA"));

        // Set the channel details into the response
        response.getEIS_RESPONSE().setChannelDetails(channelDetails);

        return response;
    }

    private ChannelDetail createChannelDetail(String flag, String mLimit, String cId, String eLimit) {
        ChannelDetail channelDetail = new ChannelDetail();
        channelDetail.setFlag(flag);
        channelDetail.setmLimit(mLimit);
        channelDetail.setcId(cId);
        channelDetail.seteLimit(eLimit);
        return channelDetail;
    }
}
