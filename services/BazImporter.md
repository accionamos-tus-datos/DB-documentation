# BazImporter
## processEvent
    BazImportHistory history = BazImportHistory.findOrCreateByFileName(key)

## startProcess
    history.startTime = LocalDateTime.now()
    history.totalRegisters = totalRegisters
    history.save(flush: true)

