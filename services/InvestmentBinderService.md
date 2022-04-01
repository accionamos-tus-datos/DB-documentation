# InvestmentBinderService
## marshallJsonToDomains
    Investment investment = Investment.findOrCreateByAggregationResponseAndActive(aggregationResponse, true)
    if(!investment.id) {
        investment.dataSaveCheckbox = json.data_save_checkbox
        investment.save(flush: true, failOnError: true)
    }
## saveCustomer
    CustomerInvestment customer = CustomerInvestment.findOrCreateByInvestmentAndFullNameAndActive(investment, json.full_name as String, true)
    customer.unnaxId = json['_id']
    customer.save(flush: true, failOnError: true)
    ...
    investment.customer = savedCustomersList[0]
    investment.save(flush:true)
    savedCustomersList << customer

    CustomerInvestment.findAllByInvestmentAndActive(investment, true).each {
        if (!savedCustomersList.contains(it)) {
            it.active = false
            it.save()
        }
    }

## savePortfolio
    Portfolio portfolio = Portfolio.findOrCreateByInvestmentAndProductNameAndActive(investment, portfolioJson.product_name, true)
    portfolio.with {
        productShortname = portfolioJson.product_shortname
        productId = portfolioJson.product_id
        currentAmount = portfolioJson.current_amount as Integer
        titlesTotal = portfolioJson.titles_total
        investedAmount = portfolioJson.invested_amount as Integer
        weightedRate = portfolioJson.weighted_rate as Double
    }
    portfolio.save(flush: true, failOnError: true)
    portfolioIds << portfolio.id
    Portfolio.findAllByInvestmentAndActive(investment, true).each {
        if (!portfolioIds.contains(it.id)) {
            it.active = false
            it.save()
        }
    }
## savePositions
        Positions positions = Positions.findOrCreateByPortfolioAndAuxKeyAndActive(portfolio, auxKey, true)
        positions.with {
            lookupId = positionJson.lookup_id
            positionId = positionJson.position_id
            series = positionJson.series
            term = positionJson.term
            acquisitionPrice = positionJson.acquisition_price as Double
            titlesNumber = positionJson.titles_number
            acquisitionRate = positionJson.acquisition_rate as Double
            investedAmount = positionJson.invested_amount as Integer
            currentAmount = positionJson.current_amount as Integer
        }
        positions.save(flush: true, failOnError: true)
        positionIds << positions.id
    
    Positions.findAllByPortfolioAndActive(portfolio, true).each {
        if (!positionIds.contains(it.id)) {
            it.active = false
            it.save()
        }
    }
## saveHistory
    History history = History.findOrCreateByDateAndInvestmentAndActive(jsonHistories.value_date, investment, true)
    history.with {
        unnaxId = historieJson['_id']
        amount = historieJson.amount as Integer
        customerId = historieJson.customer_id
        active = true
        save(flush: true)
    }
    dateList << history.date
    History.findAllByInvestmentAndActive(investment, true).each {
        if (dateList.contains(it.date)) {
            it.active = false
            it.save()
        }
    }