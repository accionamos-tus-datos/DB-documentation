# SuperCategoryService
## get
    List superCategories = SuperCategory.findAllByActive(true)
## save
    SuperCategory superCategory = new SuperCategory()
    bindData(superCategory, command)
    superCategory.save(flush: true)
## update
    SuperCategory superCategory = SuperCategory.findByIdAndActive(superCategoryId, true)
    superCategory.name = command.name
    if(command.description) {
        superCategory.description = command.description
    }
    superCategory.save(flush: true)
## delete
    SuperCategory superCategory = SuperCategory.findByIdAndActive(superCategoryId, true)
    SuperCategory scd = SuperCategory.findById(DEFAULT_SUPERCATEGORY)
    Category.findAllBySuperCategory(superCategory).each {
        it.superCategory = scd
        it.save()
    }
    superCategory.active = false
    superCategory.save(flush: true)