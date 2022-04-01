# AccountsController
## index
    
    Institution baz = Institution.findByUnnaxIdAndSourceType(SOURCE_ID_BAZ, banksId)
    
    List<AggregationResponse> aggregationResponses = AggregationResponse.findAllByUserAndEnabledAndUuidAndInstitutionNotEqual(user, true, uuid, baz)
    ...
    List<AggregationSummary> aggs = AggregationSummary.findAllByAggregationResponseInList(aggregationResponses)
## requestUpdate
    AggregationResponse ar = AggregationResponse.findByIdAndEnabledAndUuid(id, true, uuid)
    AggregationSummary aggs = AggregationSummary.findByAggregationResponse(ar)
    ..
    ar.lastRequest = LocalDateTime.now()
    ar.save(flush: true)
## deleteAggregation
## changeCredentials
    
    AggregationResponse ar = AggregationResponse.findByIdAndUuidAndEnabled(id, uuid, true)
## listCompanies
    
    Institution.findAll()
## requetsBatchUpdate
    
    Institution baz = Institution.findByUnnaxIdAndSourceType(SOURCE_ID_BAZ, banksId)
    
    List<AggregationResponse> ar = AggregationResponse.findAllByUserAndUuidAndEnabledAndToUpdateAndInstitutionNotEqual(user, uuid,true, false, baz)