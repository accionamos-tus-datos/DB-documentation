# AccountsService
## getAggregation
    
    Institution baz = Institution.findByUnnaxIdAndSourceType(SOURCE_ID_BAZ, banksId)
    List<AggregationResponse> aggregationResponses = AggregationResponse.findAllByUserAndEnabledAndUuidAndInstitutionNotEqual(user, true, uuid, baz)
    List<AggregationSummary> aggs = AggregationSummary.findAllByAggregationResponseInList(aggregationResponses)
Se podría ahorrar la consulta de la Institución para sólo realizar el listado de AggregationResponse
## getAccountList
    def cardsT = Card.findAllBySummaryAndActive(aggsum,
                    true, [sort: "paymentDueDate", order:  "desc"])
## deleteAggregation
Este se podría realizar mediante una eliminación en cascada del AggregationResponse, AggregationSummary, Account, Card y Statements

    AggregationSummary aggs = AggregationSummary.findByAggregationResponse(aResp)
    ...
    aggs.accounts.each {Account account ->
        if(account) {
            Statement.findAllByAccount(account).each {
                it.delete()
            }
        }
    }
    aggs.cards.each { Card card ->
        if(card) {
            Statement.findAllByCard(card).each {
                it.delete()
            }
        }
    }
    Customer.findAllBySummary(aggs).each {it.delete()}
    ...
    aggs.delete(flush:true, failOnError:true)
    aResp.enabled = false
    aResp.save()
## getAccountDetail
    Account account = Account.findByIdAndActive(accountId, true)
    
    Statement.findAllByAccount(account, [sort: "depositDate", order: "desc"])
## getCardDetail
    
    Card account = Card.findByIdAndActive(cardId, true)
    
    Statement.findAllByCard(account, [sort: "depositDate", order: "desc"])
## getInvestmentDetail
    
    Investment investment = Investment.findById(invId)
## getSummaryQueryInstitution
    
    AggregationResponse ar = AggregationResponse.findByIdAndUuidAndEnabled(id, uuid, true)
    
    AggregationSummary aggs = AggregationSummary.findByAggregationResponse(ar)
    
## - getCreditCardDetails
    
    List<Card> creditCardList = Card.findAllBySummaryAndActive(aggregationSummary, true, [sort: "paymentDueDate", order: "desc"])
