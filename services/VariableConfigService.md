# VariableConfigService
## getAll
    List variableConfigList = VariableConfig.findAllByActive(true)
## getConfig
    VariableConfig variableConfig = VariableConfig.findByIdAndActive(id, true)
## saveConfig
    VariableConfig variableConfig = new VariableConfig(name: vc.name, description: vc.description, value: vc.value)
    variableConfig = variableConfig.save(flush: true)
## updateConfig
    VariableConfig variableConfig = VariableConfig.findByIdAndActive(id, true)
    ...
    variableConfig.with {
        name = vc.name
        description = vc.description
        value = vc.value
    }
    variableConfig = variableConfig.save(flush: true)
## deletedConfig
    VariableConfig variableConfig = VariableConfig.findByIdAndActive(id, true)
    ...
    variableConfig.active = false
    variableConfig = variableConfig.save(flush: true)