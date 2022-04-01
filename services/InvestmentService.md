# InvestmentService
## getAllInvestments
    List<Institution> institutions = Institution.findAllBySourceType(5l)
    List<AggregationResponse> aRes  = AggregationResponse.findAllByUserAndUuidAndInstitutionInListAndEnabled(user, uuid, institutions,true)
    ...
    List<Investment> investmentList = Investment.findAllByAggregationResponseInList(aRes)
## getPortfolioDetail
    Portfolio portfolio = Portfolio.findByIdAndActive(portfolioId, true)
    AggregationResponse aRes = portfolio?.investment?.aggregationResponse
    ...
    List positionList = Positions.findAllByPortfolioAndActive(portfolio, true)
## getHistoryDetail
    Investment investment = Investment.findByIdAndActive(investmentId, true)
    AggregationResponse aRes = investment?.aggregationResponse
    ...
    List investmentList = History.findAllByInvestmentAndActive(investment, true)
## investmentToMap
    List portfolios = Portfolio.findAllByInvestmentAndActive(investment, true)
## deleteAggregation
    Investment inv = Investment.findByAggregationResponse(aResp)
    CustomerInvestment.findAllByInvestment(inv).each {
        it.delete()
    }
    History.findAllByInvestment(inv).each {
        it.delete()
    }
    Portfolio.findAllByInvestment(inv).each { Portfolio portfolio ->
        Positions.findAllByPortfolio(portfolio).each { Positions position ->
            position.delete()
        }
        portfolio.delete()
    }
    inv.delete(flush:true, failOnError:true)
    aResp.enabled = false
    aResp.save()