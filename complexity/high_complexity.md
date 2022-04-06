
## CategoryService.updateCategoriesUnnax
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