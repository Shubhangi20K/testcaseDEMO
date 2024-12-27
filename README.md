package com.example.demo.controller;

import com.example.demo.model.LimitFlagRequest;
import com.example.demo.model.LimitFlagResponse;
import com.example.demo.service.LimitFlagService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/limitflag")
public class LimitFlagController {

    @Autowired
    private LimitFlagService limitFlagService;

    @PostMapping("/inquiry")
    public ResponseEntity<LimitFlagResponse> getLimitFlag(@RequestBody LimitFlagRequest request) {
        // Call the service to get the response
        LimitFlagResponse response = limitFlagService.getLimitFlag(request);
        
        // Return the response
        return ResponseEntity.ok(response);
    }
}

