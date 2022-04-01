# BazProcessor
## processMessage
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

## saveStatementByAccount
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

## createLog
    CategorizeRequest categorizeRequest = CategorizeRequest.findOrCreateByRequestCode(requestCode)
    categorizeRequest.save(flush: true)
## updateLog 
    categorizeRequest.jobId = jobId
    categorizeRequest.save(flush: true)
## errorLog
    categorizeRequest.error = error
    categorizeRequest.save(flush: true)
## updateHistory
    BazImportHistory history = BazImportHistory.findByFileName(fileName)
    history.processedRegisters = history.processedRegisters + processedRegisters
    history.active = !(history.processedRegisters == history.totalRegisters)
    history.endTime = history.active ? null : LocalDateTime.now()
    history.save(flush: true)