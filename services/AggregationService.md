# AggregationService
## processCategorization
    
    AggregationRequest aReq = AggregationRequest.findByRequestCode(json.request_code)
## generalProcess
    
    AggregationRequest aReq = AggregationRequest.findByRequestCode(callback.trace_identifier)
    ...

    AggregationResponse aRes = aReq.aggregationResponse ?: AggregationResponse.findByTrace_identifier(
                (String) json.request_code)
    ...
    
    aRes.retry = false
    aRes.lastUpdated = LocalDateTime.now()
    aRes.save(flush: true)
## isFirst
    Boolean existsOne = Card.createCriteria().get { 
        projections { 
            countDistinct("id") 
        }
        summary { 
            aggregationResponse { 
                user { eq('id', _user.id) }
                eq('uuid', _uuid) 
            } 
        } 
    } != 0 ||
    Account.createCriteria().get {
        projections {
            countDistinct("id")
        }
        summary {
            aggregationResponse {
                user { eq('id', _user.id) }
                eq('uuid', _uuid)
            }
        }
    }  != 0 ||
    Retail.createCriteria().get { 
        projections { 
            countDistinct("id")
        }
        aggregationResponse { 
            user { eq('id', _user.id) }
            eq('uuid', _uuid) 
        }
    }  != 0 ||
    Investment.createCriteria().get { 
        projections { 
            countDistinct("id")
        }
        aggregationResponse { 
            user { eq('id', _user.id) }
            eq('uuid', _uuid) 
        }
    }
