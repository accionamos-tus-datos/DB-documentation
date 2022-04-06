## AccountController.index
    
    Institution baz = Institution.findByUnnaxIdAndSourceType(SOURCE_ID_BAZ, banksId)
    
    List<AggregationResponse> aggregationResponses = AggregationResponse.findAllByUserAndEnabledAndUuidAndInstitutionNotEqual(user, true, uuid, baz)
    ...
    List<AggregationSummary> aggs = AggregationSummary.findAllByAggregationResponseInList(aggregationResponses)
## AccountController.requestUpdate
    AggregationResponse ar = AggregationResponse.findByIdAndEnabledAndUuid(id, true, uuid)
    AggregationSummary aggs = AggregationSummary.findByAggregationResponse(ar)
    ..
    ar.lastRequest = LocalDateTime.now()
    ar.save(flush: true)
## AccountController.changeCredentials
    AggregationResponse ar = AggregationResponse.findByIdAndUuidAndEnabled(id, uuid, true)
## AccountController.listCompanies
    Institution.findAll()
## AccountController.requetsBatchUpdate
    Institution baz = Institution.findByUnnaxIdAndSourceType(SOURCE_ID_BAZ, banksId)
    
    List<AggregationResponse> ar = AggregationResponse.findAllByUserAndUuidAndEnabledAndToUpdateAndInstitutionNotEqual(user, uuid,true, false, baz)
    //Puede igualarse con el metodo index, a comprobar
## AccountsService.getAggregation
    Institution baz = Institution.findByUnnaxIdAndSourceType(SOURCE_ID_BAZ, banksId)
    List<AggregationResponse> aggregationResponses = AggregationResponse.findAllByUserAndEnabledAndUuidAndInstitutionNotEqual(user, true, uuid, baz)
    List<AggregationSummary> aggs = AggregationSummary.findAllByAggregationResponseInList(aggregationResponses)
    //similar a AccountsController.index

## AccountsService.getAccountList
    aggsum.accounts...
    def cardsT = Card.findAllBySummaryAndActive(aggsum,
                    true, [sort: "paymentDueDate", order:  "desc"])
## AccountService.getAccountDetail
    Account account = Account.findByIdAndActive(accountId, true)
    
    Statement.findAllByAccount(account, [sort: "depositDate", order: "desc"])
    
## AccountService.getCardDetail
    Card account = Card.findByIdAndActive(cardId, true)
    
    Statement.findAllByCard(account, [sort: "depositDate", order: "desc"])
## AccountService.getSummaryQueryInstitution
    
    AggregationResponse ar = AggregationResponse.findByIdAndUuidAndEnabled(id, uuid, true)
    
    AggregationSummary aggs = AggregationSummary.findByAggregationResponse(ar)
    
## - AccountService.getCreditCardDetails
    
    List<Card> creditCardList = Card.findAllBySummaryAndActive(aggregationSummary, true, [sort: "paymentDueDate", order: "desc"])
## AggregationBindersService.marshallJsonToDomains
    
    AggregationRequest ar = AggregationRequest.findByRequestCode((String) json.request_code)
    
    AggregationLog al = AggregationLog.findByRequestCode((String) json.request_code)
    
    AggregationSummary aggs = AggregationSummary.findByAggregationResponse( aRes )
    ...
    if(aRes.toUpdate) {
        aRes.toUpdate = false
    }
    aRes.save(flush: true)
    ...
    
    al.with{
        message = json.message
        code = json.code
        status = AggregationStatus.INCOMPLETE_DATA
        save()
    }
## AggregationBindersService.parseStatements
    
    [account: Account.findByAccountNoAndSummaryAndActive((String) statement.account, summary, true),
    card: Card.findByCardNoAndSummaryAndActive((String) statement.partial_card_number, summary, true)]
    ...
    List<Statement> sttmtsDepositDate = Statement.findAllByAccountAndCardAndDepositDate(relatedAccount, relatedCard, relatedDepositDate)
## AggregationBindersService.getCategory
    
    Category cat = Category.findOrCreateByUnnaxId((Long) categ.id)
    ...
        cat.title = categ.title?.take(100)
        cat.superCategory = superCategory
        cat.state = true
        cat.parentCategory = parentCategory
        cat.save(flush:true)

## AggregationBindersService.saveStatement
    statement.with {
        card = rCard
        account = rAccount
        unnaxId = json["_id"] as Long
        ammount = (Long) json.amount
        concepts = ""
        json.concepts?.eachWithIndex { conceptPart, index ->
            concepts += conceptPart
        }
        if(concepts.length()>255){
            concepts = "${concepts.substring(0,252)}..."
        }
        currency = json.currency
        depositDate = rDepositDate
        importDate = json.import_date
        settlement = json.settlement
    }
    if (!statement.category.computable || !matchesCurrency(statement.currency)) {
        statement.computable = false
    }
    
    Boolean success = statement.save(flush:true)
## AggregationBindersService.updateLastMovement
relatedAccount puede ser de tipo Account o Card
    
    relatedAccount.lastMovement = statementDate
    Boolean success = relatedAccount.save(flush:true)
## AggregationBindersService.aggregationLogger
    
    al.with{
        customers = percentage
        accounts = percentage
        statements = percentage
        cards = percentage
        cardStatements = percentage
        Map<String, Double> stats = percentageObjects
        completed = stats.general
        objectsSaved = stats.saved
        status = AggregationStatus.SUCCESSFUL_PROCESS
        save()
    }

## AggregationService.processCategorization
    
    AggregationRequest aReq = AggregationRequest.findByRequestCode(json.request_code)
## AggregationService.generalProcess
    
    AggregationRequest aReq = AggregationRequest.findByRequestCode(callback.trace_identifier)
    ...

    AggregationResponse aRes = aReq.aggregationResponse ?: AggregationResponse.findByTrace_identifier(
                (String) json.request_code)
    ...
    
    aRes.retry = false
    aRes.lastUpdated = LocalDateTime.now()
    aRes.save(flush: true)

## AggregationWidgetService.getWidgetUrl
    
    new AggregationRequest(requestCode: rc, user: user, uuid: uuid, sourceType: wic.sourceType).save()
## AggregationWidgetService.getWidgetUrlUpdate
    
    new AggregationRequest(requestCode: rc, user: ar.user, aggregationResponse: ar, uuid: uuid, sourceType: ar.institution.sourceType).save()

## BazImporter.processEvent
    BazImportHistory history = BazImportHistory.findOrCreateByFileName(key)

## BazImporter.startProcess
    history.startTime = LocalDateTime.now()
    history.totalRegisters = totalRegisters
    history.save(flush: true)

## BazProcessor.processMessage
    Institution institution = Institution.findByUnnaxIdAndSourceType(institutionId, sourceType)
    User user = User.findByUsername(username)
    ...
        AggregationResponse aggregationResponse
        AggregationSummary aggregationSummary
        aggregationResponse = AggregationResponse.findOrCreateByUserAndUuidAndInstitution(user, bazUser, institution)
        if (!aggregationResponse.id || !aggregationResponse.enabled) {
            aggregationResponse.enabled = true
            aggregationResponse.save(flush: true)
        }
        aggregationSummary = AggregationSummary.findOrCreateByAggregationResponse(aggregationResponse)
        aggregationSummary.save(flush: true)
    ...
    ...
    AggregationRequest request
    request = new AggregationRequest(requestCode: rc, user: user, uuid: '', sourceType: 1)
    request.save(flush: true)

## BazProcessor.saveStatementByAccount
    Account account = null
    BazStatement last = statements.max { it.date }
    account = Account.findOrCreateByAccountNoAndSummaryAndActiveAndCurrencyAndName(accountNo, aggregationSummary, true, currency, name)
    account.with {
        balance = (Long) last.balance
        lastMovement = (LocalDate) last.date
        save(flush: true)
    }
    ...
    Statement.withNewTransaction {
    statements.eachWithIndex { BazStatement statement, Long index ->
        Statement s = new Statement(index, account, statement)
        /*
            Statement(Long unnaxId, Account account, BazStatement data) {
                this.unnaxId = unnaxId
                this.ammount = (Long) data.amount
                this.account = account
                this.concepts = ((String) data.description).take(252) ?: ''
                this.currency = data.currency
                this.depositDate = (LocalDate) data.date
                this.importDate = LocalDateTime.now()
                this.operationType = data.operationCode
            }
        */
        if(s.save(flush: true)) {...}
    }

## BazProcessor.createLog
    CategorizeRequest categorizeRequest = CategorizeRequest.findOrCreateByRequestCode(requestCode)
    categorizeRequest.save(flush: true)
## BazProcessor.updateLog 
    categorizeRequest.jobId = jobId
    categorizeRequest.save(flush: true)
## BazProcessor.errorLog
    categorizeRequest.error = error
    categorizeRequest.save(flush: true)
## BazProcessor.updateHistory
    BazImportHistory history = BazImportHistory.findByFileName(fileName)
    history.processedRegisters = history.processedRegisters + processedRegisters
    history.active = !(history.processedRegisters == history.totalRegisters)
    history.endTime = history.active ? null : LocalDateTime.now()
    history.save(flush: true)
## CategoryService.updateCategoryNameToSpanish
    Category category = Category.findByUnnaxId(id)
    category.spanishTitle = spanishTitle
    category.save(flush: true)
    
## CategoryService.updateCategoriesWebhook
    
    Category category = Category.findByUnnaxId(cat.id)
    
    Category parentCat = command.parent_id ? Category.findByUnnaxId(command.parent_id) : null
    
    SuperCategory sc = category.superCategory ?: SuperCategory.findById(DEFAULT_SUPERCATEGORY)
    category.with {
        unnaxId = command.id
        title = command.title
        state = command.state
        type = command.type
        parentCategory = parentCat
        superCategory = sc
    }
    
    boolean success = category.save(flush:true)

## InvestmentBinderService.marshallJsonToDomains
    Investment investment = Investment.findOrCreateByAggregationResponseAndActive(aggregationResponse, true)
    if(!investment.id) {
        investment.dataSaveCheckbox = json.data_save_checkbox
        investment.save(flush: true, failOnError: true)
    }
## InvestmentBinderService.saveCustomer
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

## InvestmentService.getAllInvestments
    List<Institution> institutions = Institution.findAllBySourceType(5l)
    List<AggregationResponse> aRes  = AggregationResponse.findAllByUserAndUuidAndInstitutionInListAndEnabled(user, uuid, institutions,true)
    ...
    List<Investment> investmentList = Investment.findAllByAggregationResponseInList(aRes)
## InvestmentService.getPortfolioDetail
    Portfolio portfolio = Portfolio.findByIdAndActive(portfolioId, true)
    AggregationResponse aRes = portfolio?.investment?.aggregationResponse
    ...
    List positionList = Positions.findAllByPortfolioAndActive(portfolio, true)
## InvestmentService.getHistoryDetail
    Investment investment = Investment.findByIdAndActive(investmentId, true)
    AggregationResponse aRes = investment?.aggregationResponse
    ...
    List investmentList = History.findAllByInvestmentAndActive(investment, true)
## InvestmentService.investmentToMap
    List portfolios = Portfolio.findAllByInvestmentAndActive(investment, true)

## MovementHistoryService.updateOutcomeAndIncomeHistory
    VariableConfig variableConfig = VariableConfig.findByNameAndActive('updateHistory', true)
    ...
    MovementHistory movementHistory = MovementHistory.findByUserAndUuidAndYearAndMonthAndActive(user, uuid, date.minusMonths(i).year, monthValue.id, true)
    movementHistory.with {
        income = history.income as Double
        outcome = history.outcome as Double
        balance = history.balance as Double
    }
    movementHistory = movementHistory.save(flush: true)
## MovementHistoryService.searchHistory
    MovementHistory.findByUserAndUuidAndYearAndMonthAndActive(user, uuid, year, month, true)
## MovementHistoryService.saveHistory
    MovementHistory movementHistory = new MovementHistory(
            period: history.period,
            monthName: history.monthName,
            uuid: uuid,
            income: history.income as Double,
            outcome: history.outcome as Double,
            balance: history.balance as Double,
            month: month,
            year: year,
            user: user
    )
    movementHistory = movementHistory.save(flush: true)

## StatementService.getByCategoryInPeriod
    Category _category = Category.findByUnnaxId(categoryId)


## TokenizationService.updateDataFromTokenization
    Institution baz = Institution.findByUnnaxIdAndSourceType(SOURCE_ID_BAZ, banksId)
    AggregationResponse.findAllByEnabledAndRetryAndToUpdateAndInstitutionNotEqual(true, !first, false, baz)
## TokenizationService.processError
    AggregationRequest aReq = AggregationRequest.findByRequestCode(command.trace_identifier)
    ...
    AggregationResponse aRes = aReq.aggregationResponse
    aRes.retry = retry
    aRes.save(flush: true)
## TokenizationService..preSendTokenization
    new AggregationRequest(requestCode: requestCode, user: ar.user, aggregationResponse: ar , uuid: ar.uuid, sourceType: ar.institution.sourceType).save(flush:true)
    new AggregationLog(requestCode: requestCode, user: ar.user, institution: ar.institution, status: AggregationStatus.UPDATE_START).save()
## TokenizationService.invalidAggregation(AggregationResponse ar )
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
## UserService.validate
    User user = User.findByUsername(vcc.username)
    ...
    user.apiKey = apiK
    ...
    Token tkn = Token.findByTokenAndUserAndActive(verificationCode, user, true)
    ...
    tkn.with {
        user.enabled = true
        user.save()

        active = false
        token = "0000"
        save()
    }
## UserService.sendRecoveryToken
    User user = User.findByUsername(username)
    ...
    Token token = Token.findOrCreateByUserAndTokenType(user, TokenType.RECOVERY)
    token.active = true
    token.token = token.generateOtp()
    token.save(flush: true)
## UserService.resetPassword
    User user = User.findByUsername(rpc.username)
    ...
    Token tkn = Token.findByTokenAndUserAndActive(verificationCode, user, true)
    tkn.with {
        user.password = rpc.password
        user.save()

        active = false
        token = "0000"
        save()
    }
## UserService.resendTokenConfirmation
    User user = User.findByUsername(username)
    ...
    Token token = Token.findOrCreateByUserAndTokenType(user, TokenType.CONFIRMATION)
    token.active = true
    token.token = token.generateOtp()
    token.save(flush: true)
## UserService.validateVerifiedAccount
    User user = User.findByUsernameAndEnabled(username, true)
## UserService.updateApiKey
    user.apiKey = generateRequestHash()
    user.save(flush: true)

## VariableConfigService.getAll
    List variableConfigList = VariableConfig.findAllByActive(true)
## VariableConfigService.getConfig
    VariableConfig variableConfig = VariableConfig.findByIdAndActive(id, true)
## VariableConfigService.saveConfig
    VariableConfig variableConfig = new VariableConfig(name: vc.name, description: vc.description, value: vc.value)
    variableConfig = variableConfig.save(flush: true)
## VariableConfigService.updateConfig
    VariableConfig variableConfig = VariableConfig.findByIdAndActive(id, true)
    ...
    variableConfig.with {
        name = vc.name
        description = vc.description
        value = vc.value
    }
    variableConfig = variableConfig.save(flush: true)
## VariableConfigService.deletedConfig
    VariableConfig variableConfig = VariableConfig.findByIdAndActive(id, true)
    ...
    variableConfig.active = false
    variableConfig = variableConfig.save(flush: true)