# Before
## 1. Find with inner join

#### CODE
    List<Institution> institutions = Institution.findAllBySourceType(3l)
    List<AggregationResponse> responses  = AggregationResponse.findAllByUserAndUuidAndEnabledAndInstitutionInList(user, uuid, true, institutions)
    if(!responses) {
        return [noOfRetails: 0, globalBalance: 0, globalDebt: 0, retails: []]
    }
    List<Retail> retails = Retail.findAllByAggregationResponseInList(responses)
    
#### HIBERNATE CALL
    Hibernate: select this_.id as id1_16_0_, this_.version as version2_16_0_, this_.date_created as date_cre3_16_0_, this_.alias as alias4_16_0_, this_.last_updated as last_upd5_16_0_, this_.name as name6_16_0_, this_.active as active7_16_0_, this_.unnax_id as unnax_id8_16_0_, this_.country as country9_16_0_, this_.source_type as source_10_16_0_ from institution this_ where this_.source_type=?
    Hibernate: select this_.id as id1_4_0_, this_.version as version2_4_0_, this_.environment as environm3_4_0_, this_.date as date4_4_0_, this_.last_request as last_req5_4_0_, this_.date_created as date_cre6_4_0_, this_.uuid as uuid7_4_0_, this_.last_updated as last_upd8_4_0_, this_.token_key as token_ke9_4_0_, this_.signature as signatu10_4_0_, this_.triggered_event as trigger11_4_0_, this_.retry as retry12_4_0_, this_.response_id as respons13_4_0_, this_.service as service14_4_0_, this_.user_id as user_id15_4_0_, this_.to_update as to_upda16_4_0_, this_.token_id as token_i17_4_0_, this_.trace_identifier as trace_i18_4_0_, this_.enabled as enabled19_4_0_, this_.institution_id as institu20_4_0_, this_.data as data21_4_0_ from aggregation_response this_ where this_.user_id=? and this_.uuid=? and this_.enabled=? and this_.institution_id in (?, ?)
    Hibernate: select this_.id as id1_22_0_, this_.version as version2_22_0_, this_.date_created as date_cre3_22_0_, this_.aggregation_response_id as aggregat4_22_0_, this_.last_updated as last_upd5_22_0_, this_.active as active6_22_0_ from retail this_ where this_.aggregation_response_id in (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)

## 2.- Create or update based on a relation

#### CODE
    Retail retail = Retail.findOrCreateByAggregationResponse(aResponse)
    if(!retail.id) {
        retail.save(flush: true)
    }

#### HIBERNATE CALL
###### INSERT

    Hibernate: select this_.id as id1_22_0_, this_.version as version2_22_0_, this_.date_created as date_cre3_22_0_, this_.aggregation_response_id as aggregat4_22_0_, this_.last_updated as last_upd5_22_0_, this_.active as active6_22_0_ from retail this_ where this_.aggregation_response_id=? limit ?
    Hibernate: insert into retail (version, date_created, aggregation_response_id, last_updated, active) values (?, ?, ?, ?, ?)

###### SELECT

    Hibernate: select this_.id as id1_22_0_, this_.version as version2_22_0_, this_.date_created as date_cre3_22_0_, this_.aggregation_response_id as aggregat4_22_0_, this_.last_updated as last_upd5_22_0_, this_.active as active6_22_0_ from retail this_ where this_.aggregation_response_id=? limit ?

    