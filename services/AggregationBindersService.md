# AggregationBindersService
## marshallJsonToDomains
    
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
## parseCustomers
    
    Customer customer = Customer.findOrCreateByNameAndSummary(it, aggs)
    ...
    customer.save(flush:true)
    ...
    
    aggs.customer = savedCustomersList[0]
    Boolean success = aggs.save(flush:true)
## parseAccounts
    
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
## parseStatements
    
    [account: Account.findByAccountNoAndSummaryAndActive((String) statement.account, summary, true),
    card: Card.findByCardNoAndSummaryAndActive((String) statement.partial_card_number, summary, true)]
    ...
    List<Statement> sttmtsDepositDate = Statement.findAllByAccountAndCardAndDepositDate(relatedAccount, relatedCard, relatedDepositDate)
## getCategory
    
    Category cat = Category.findOrCreateByUnnaxId((Long) categ.id)
    ...
        cat.title = categ.title?.take(100)
        cat.superCategory = superCategory
        cat.state = true
        cat.parentCategory = parentCategory
        cat.save(flush:true)

## saveStatement
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
## updateLastMovement
relatedAccount puede ser de tipo Account o Card
    
    relatedAccount.lastMovement = statementDate
    Boolean success = relatedAccount.save(flush:true)
## parrseCards
    
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
## aggregationLogger
    
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
## updateCategoriesFromUnnax
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
## defaultCategoryByStatement
    Long categoryId = DEFAULT_CATEGORY_OUTCOME
    if (statement.card) {
        categoryId = (statement.ammount < 0) ? DEFAULT_CATEGORY_OUTCOME : DEFAULT_CATEGORY_INCOME
    } else if (statement.account) {
        categoryId = (statement.ammount > 0) ? DEFAULT_CATEGORY_OUTCOME : DEFAULT_CATEGORY_INCOME
    }
    
    Category.findByUnnaxId(categoryId)
## callStoredProcedure

    sql.call ("CALL sp_categorization_business_rules(?,?)",[id, uuid])
