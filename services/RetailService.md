# RetailService
## getAll
    List<Institution> institutions = Institution.findAllBySourceType(3l)
    List<AggregationResponse> responses  = AggregationResponse.findAllByUserAndUuidAndEnabledAndInstitutionInList(user, uuid, true, institutions)ç
    ...
    List<Retail> retails = Retail.findAllByAggregationResponseInList(responses)
## getShoppingItems
    ShoppingRetail shopping = ShoppingRetail.findById(shoppingId)
    AggregationResponse aRes = shopping?.customerRetail?.retail?.aggregationResponse
## find
    Retail retail = Retail.findById(id)
    AggregationResponse aRes = retail?.aggregationResponse
## deleteAggregation
    Retail retail = Retail.findByAggregationResponse(aResp)
    ...
    retail.delete()
    aResp.enabled = false
    aResp.save()