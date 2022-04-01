# StatementService
## getByCategoryInPeriod
    Category _category = Category.findByUnnaxId(categoryId)
## getByCategoryAndPeriodAndCardType
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
## updateStatement 
    Category entry = Category.findByUnnaxId(it.key)
    Category category = entry.parentCategory
    List<Long> ids = it.value.entry_id.collect{it as Long}
    List<Statement> statements = Statement.findAllByIdInList(ids)
    if(statements) {
        statements.each {s ->
            s.category = category ?: entry
            s.subCategory = category != null ? entry : null
        }
        Statement.saveAll(statements)
        saved.addAll(statements)
    }
## updateLastMovement
    //parent puede ser Card o Account
    if(parent.lastMovement.isBefore(lastMovement)) {
        parent.lastMovement = lastMovement
        parent.save(failOnError: true)
    }
## updateAllAggregation
    Institution baz = Institution.findByUnnaxIdAndSourceType(SOURCE_ID_BAZ, banksId)
    List<AggregationResponse> ars = AggregationResponse.findAllByInstitution(baz)
    ...
    ars[(init)..(finish)].each { it.lastRequest = yesterday }
    AggregationResponse.saveAll(ars[(init)..(finish)])