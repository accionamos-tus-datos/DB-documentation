## AccountsService.deleteAggregation
    Este se podría realizar mediante una eliminación en cascada del    AggregationResponse, AggregationSummary, Account, Card y Statements

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
## AggregationBindersService.parseCustomers
    Customer customer = Customer.findOrCreateByNameAndSummary(it, aggs)
    ...
    customer.save(flush:true)
    ...
    
    aggs.customer = savedCustomersList[0]
    Boolean success = aggs.save(flush:true)
## AggregationBindersService.parseAccounts
    
    Account account = Account.findOrCreateByAccountNoAndSummaryAndActive((String) accountJson['_id'], aggs, true)
    ...
    
    account.with {
        customers = customerList
        currency = accountJson.currency
        balance = (Long) accountJson.balance
        name = accountJson.name
    }
    ...
    
    aggs.addToAccounts(account)
    ....
    
    Boolean success = aggs.save(flush:true) //Aquí se le hace save a todas las cuentas que se crearon o actualizarón
    ...
    //Proceso para desactivar cuentas que no se encuentren activas en el banco
    Account.findAllBySummaryAndActive(aggs, true).each {
        if(!accountsIds.contains(it.accountNo)) {
            it.active = false
            it.save()
        }
    }
## AggregationBindersService.parseCards
    
    Card card = Card.findOrCreateByCardNoAndSummaryAndActive(cardJson.partial_card_number, aggs, true)
    ...
    
    card.with {
        unnaxId = (Long) cardJson['_id']
        customers = customerList
        currency = cardJson.currency
        creditAvailable = (Long) cardJson.credit_available
        creditLimit = (Long) cardJson.credit_limit
        name = ((String)cardJson.name).replace("\u00ae","")
        account = cardJson.account
        cardNo = cardJson.partial_card_number
        activeUnnax = cardJson.active
        periodEndingDate = cardJson.period_ending_date
        paymentDueDate = cardJson.payment_due_date
        minimumPayment = cardJson.minimum_payment as Long
        paymentToAvoidInterest = cardJson.payment_to_avoid_interest as Long
        creditBalance = (Long) cardJson.credit_balance
    }
    ...
    
    aggs.addToCards(card)
    ...
    
    Boolean success = aggs.save(flush:true) //Aquí es dónde se salvan todas las Cards actualizadas o creadas
    
    Card.findAllBySummaryAndActive(aggs, true).each {
        if(!cardsIds.contains(it.cardNo)) {
            it.active = false
            it.save()
        }
    }
## AggregationBindersService.updateCategoriesFromUnnax
Esta función junto con ls siguiente podrían colocarse dentro del mismo SP para simplificar

    
    Statement statementToUpdate = Statement.findByCardAndAccountAndDepositDateAndUnnaxId(relatedCard, relatedAccount, relatedDepositDate, statement['_id'] as Long)
    ...
    
    SuperCategory defaultSC = SuperCategory.findById(DEFAULT_SUPERCATEGORY)
    statementDataFromUnnax?.category?.hierarchy?.eachWithIndex { Map categ, index ->
        if(index == 0) {
            cat = getCategory(categ, defaultSC, null)
        }
        if(index == 1) {
            subCat = getCategory(categ, defaultSC, cat)
        }
    }
    statement.category = cat ?: defaultCategoryByStatement(statement)
    statement.subCategory = subCat
    ...
    
    Boolean success = statementToUpdate.save(flush: true)
## AggregationBindersService.defaultCategoryByStatement
    Long categoryId = DEFAULT_CATEGORY_OUTCOME
    if (statement.card) {
        categoryId = (statement.ammount < 0) ? DEFAULT_CATEGORY_OUTCOME : DEFAULT_CATEGORY_INCOME
    } else if (statement.account) {
        categoryId = (statement.ammount > 0) ? DEFAULT_CATEGORY_OUTCOME : DEFAULT_CATEGORY_INCOME
    }
    
    Category.findByUnnaxId(categoryId)
## AggregationService.isFirst
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
## BankService.saveInstitution
    
    def listDisable = Institution.findAllBySourceTypeAndUnnaxIdNotInListAndActive(sourceType, list?.source_id, true)
        
        listDisable.each {
            it.active = false
            it.save(flush:true, failOnError:true)
        }
        list?.each {
            
            Institution b = Institution.findOrCreateByUnnaxIdAndSourceType(it.source_id, sourceType)
            b.active = true
            b.name = it.name
            b.unnaxId = it.source_id
            b.country = it.country
            b.alias = it?.group_name
            
            b.save(flush:true, failOnError:true)
        }
## CategoryService.filterStatements
    List<Statement> = statementList = statementCriteria.list {
        if (isOutcome) {
            card { summary { aggregationResponse {
                eq('user', user)
                eq('uuid', uuid)
                eq('enabled', true)
            } } }
        } else {
            account { summary { aggregationResponse {
                eq('user', user)
                eq('uuid', uuid)
                eq('enabled', true)
            } } }
        }
        category {
            'in'('type', (!isOutcome && forceOutcome) ? [0, 2]: [1, 3])
        }
        eq("computable", true)
        between('depositDate', from, to)
    } + (isOutcome ? filterStatements(uuid, user, year, month, isOutcome: false, forceOutcome: false) : [])
## CategoryService.obtainSumCurrentAmountsDebitCards
    List result = Account.createCriteria().get() {
        summary {
            aggregationResponse {
                eq('user', user)
                eq('uuid', uuid)
                eq('enabled', true)
            }
        }
        eq('active', true)
        projections {
            sum('balance')
            countDistinct('id')
            summary {
                aggregationResponse {
                    max('lastRequest')
                }
            }
        }
    } as List
## InvestmentBinderService.savePortfolio
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
## InvestmentBinderService.savePositions
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
## InvestmentBinderService.saveHistory
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
## InvestmentService.deleteAggregation
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
## StatementService.getByCategoryAndPeriodAndCardType
    Statement.createCriteria().list {
        if (!outcomes) {
            account { summary { aggregationResponse {
            eq('user', user)
            eq('uuid', uuid)
            eq('enabled', true)
            } } }
        } else {
            card { summary { aggregationResponse {
            eq('user', user)
            eq('uuid', uuid)
            eq('enabled', true)
            } } }
        }
        category {
            eq('unnaxId', categoryId)
        }
        eq("computable", true)
        between('depositDate', from, to)
    } as List) + (outcomes ? getByCategoryAndPeriodAndCardType(categoryId, from, to, uuid, user, outcomes: false) : []

## TokenizationService.saveTokenization
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

## UserService.register
    User user = new User(
            username: command.username,
            password: command.password,
            email: command.email
    )
    ...
    User toValidateExistance = User.findByUsername(user.username)
    user = user.save()
    ...
    Role role = Role.findByAuthority("ROLE_USER")
    UserRole ur = UserRole.create(user, role).save()
    Token tkn = new Token(user: user, tokenType: TokenType.CONFIRMATION).save()
    if (!emailSent) {
        tkn.delete(flush:true, failOnError:true)
        ur.delete(flush:true, failOnError:true)
        user.delete(flush:true, failOnError:true)
    }