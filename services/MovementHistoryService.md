# MovementHistoryService
## updateOutcomeAndIncomeHistory
    VariableConfig variableConfig = VariableConfig.findByNameAndActive('updateHistory', true)
    ...
    MovementHistory movementHistory = MovementHistory.findByUserAndUuidAndYearAndMonthAndActive(user, uuid, date.minusMonths(i).year, monthValue.id, true)
    movementHistory.with {
        income = history.income as Double
        outcome = history.outcome as Double
        balance = history.balance as Double
    }
    movementHistory = movementHistory.save(flush: true)
## searchHistory
    MovementHistory.findByUserAndUuidAndYearAndMonthAndActive(user, uuid, year, month, true)
## saveHistory
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