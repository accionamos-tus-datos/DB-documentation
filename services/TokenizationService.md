# TokenizationService
## saveTokenization
    AggregationRequest aRequest = AggregationRequest.findByRequestCode(dto.trace_identifier)
    ...
    new AggregationLog(requestCode:dto.trace_identifier, user : aRequest.user,
            institution: Institution.findByUnnaxIdAndSourceType(sourceId, sourceType), status: AggregationStatus.TOKENIZED_CREDENTIALS).save()
    AggregationResponse item = AggregationResponse.findOrCreateByUserAndTrace_identifierAndUuid(aRequest.user, dto.trace_identifier, aRequest.uuid)

    item.institution = Institution.findByUnnaxIdAndSourceType(sourceId, sourceType)
    item.date = dto.date
    item.triggered_event = dto.triggered_event
    item.environment = dto.environment
    item.tokenKey = dto.data.token_key
    item.tokenId = dto.data.token_id
    item.save(flush: true)
    
    AggregationSummary aggs
    if(aRequest.aggregationResponse){
        AggregationResponse oldAR = aRequest.aggregationResponse
        aggs = AggregationSummary.findByAggregationResponse( oldAR )
        oldAR.enabled = false
        oldAR.save()
    else {
        aggs = new AggregationSummary()
    }
    aRequest.aggregationResponse = item
    aRequest.save()
    aggs.aggregationResponse = item
    aggs = aggs.save()
## updateDataFromTokenization
    Institution baz = Institution.findByUnnaxIdAndSourceType(SOURCE_ID_BAZ, banksId)
    AggregationResponse.findAllByEnabledAndRetryAndToUpdateAndInstitutionNotEqual(true, !first, false, baz)
## processError
    AggregationRequest aReq = AggregationRequest.findByRequestCode(command.trace_identifier)
    ...
    AggregationResponse aRes = aReq.aggregationResponse
    aRes.retry = retry
    aRes.save(flush: true)
## preSendTokenization
    new AggregationRequest(requestCode: requestCode, user: ar.user, aggregationResponse: ar , uuid: ar.uuid, sourceType: ar.institution.sourceType).save(flush:true)
    new AggregationLog(requestCode: requestCode, user: ar.user, institution: ar.institution, status: AggregationStatus.UPDATE_START).save()
## invalidAggregation(AggregationResponse ar )
    AggregationLog al = AggregationLog.findByRequestCode( ar.trace_identifier )
    if( al ){
        al.status = AggregationStatus.FAIL_PROCESS
        al.save(flush: true)
    }
    ar.with {
        toUpdate = true
        lastRequest = LocalDateTime.now()
        save(flush: true)
    }