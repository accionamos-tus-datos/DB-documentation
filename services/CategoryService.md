# CategoryService
## getBySupercategory
    SuperCategory sc = SuperCategory.findById(superCategoryId)
## updateSuperCategory
    
    Category category = Category.findById(categoryId)
    if(category.superCategory.id != superCategoryId) {
        
        SuperCategory superCategory = SuperCategory.findById(superCategoryId)
        category.superCategory = superCategory
        
        category.save(flush: true)
    }
## getCategories
    //Con un stored procedure se podría tomar la decisión internamente
    
    totalCategories = Category.countBySuperCategory(sc)
    ||
    totalCategories = Category.count()
    ...
    List categories = catCriteria.list(max: pagination.max, offset: pagination.offset) {
        if(sc) {
            eq("superCategory", sc)
        }
    }
## updateCategoriesUnnax
    Category category = Category.findOrCreateByUnnaxId(cat.id as Integer)
    if (category.id) {
        if(!category.type) {
            category.type = cat.type as Integer
            category.state = cat.state as Boolean
            if(!category.save(flush: true)){
                throw new ConnectionException("categories.unnax.update.fail")
            }

        } else {
            if(category.state != cat.state as Boolean) {
                category.state = cat.state as Boolean
                if(!category.save(flush: true)){
                    throw new ConnectionException("categories.unnax.update.fail")
                }
            }
        }
    } else {
        category.with {
            type = cat.type as Integer
            title = cat.title
            state = cat.state as Boolean
            superCategory = SuperCategory.findById(DEFAULT_SUPERCATEGORY)
        }
        if(!category.save(flush: true)){
            throw new ConnectionException("categories.unnax.update.fail")
        }
    }
    if(cat.children) {
        cat.children.each { Map subCat ->
            Category subCategory = Category.findOrCreateByUnnaxId(subCat.id as Integer)
            if (subCategory.id) {
                if(!subCategory.type) {
                    subCategory.with {
                        type = subCat.type as Integer
                        state = subCat.state as Boolean
                        parentCategory = category
                    }
                    if(!subCategory.save(flush: true)){
                        throw new ConnectionException("categories.unnax.update.fail")
                    }
                } else {
                    if(!subCategory.parentCategory || subCategory.state != subCat.state as Boolean) {
                        subCategory.parentCategory = category
                        subCategory.state = subCat.state as Boolean
                        if(!subCategory.save(flush: true)){
                            throw new ConnectionException("categories.unnax.update.fail")
                        }
                    }
                }
            } else {
                subCategory.with {
                    type = subCat.type as Integer
                    title = subCat.title
                    state = subCat.state as Boolean
                    parentCategory = category
                    superCategory = SuperCategory.findById(DEFAULT_SUPERCATEGORY)
                }
                if(!subCategory.save(flush: true)){
                    throw new ConnectionException("categories.unnax.update.fail")
                }
            }
        }
    }
## filterStatements
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
## obtainSumCurrentAmountsDebitCards
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
## updateCategoryNameToSpanish
    Category category = Category.findByUnnaxId(id)
    category.spanishTitle = spanishTitle
    category.save(flush: true)
    
## updateCategoriesWebhook
    
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