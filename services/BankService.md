# BankService
## saveInstitution
    
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
