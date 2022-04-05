# After

## 1. Find with inner join

#### CODE
    List<Retail> retails = retailRepositoryService.findAllByUserAndUuid(user.id, uuid)
    if(!retails) {
        return [noOfRetails: 0, globalBalance: 0, globalDebt: 0, retails: []]
    }

    /*findAllByUserAndUuid(user, uuid)*/
    callThroughNativeQuery("CALL SP_get_retails_by_user(:user, :uuid)") {
        NativeQuery nq ->
            nq.resultTransformer = retailTransformer
            nq.addEntity('r', Retail)
            nq.addEntity('ar', AggregationResponse)
            nq.setLong('user', user)
            nq.setString('uuid', uuid)
            nq.list()
    } as List

#### HIBERNATE CALL
    Hibernate: CALL SP_get_retails_by_user(?, ?)

#### SP
    PROCEDURE SP_get_retails_by_user(IN _user BIGINT, IN _uuid VARCHAR(255))
    BEGIN
        SELECT r.*, ar.*
        FROM retail AS r
        INNER JOIN aggregation_response AS ar
            ON r.aggregation_response_id = ar.id
            AND ar.user_id = _user
            AND ar.uuid = _uuid
            AND ar.enabled = true
        INNER JOIN institution AS i
            ON i.id = ar.institution_id
            AND i.source_type = 3;
    END

## 2.- Create or update based on a relation

#### CODE
    Retail retail = retailRepositoryService.findOrCreateByAggregationResponse(aResponse)
    if(!retail?.id) {
        return false
    }

    /*findOrCreateByAggregationResponse(response)*/
    Retail retail = callThroughNativeQuery("CALL SP_retail_find_create_by_response(:response, :createdAt)") {
        NativeQuery nq ->
            nq.addEntity(Retail)
            nq.setLong('response', response.id)
            nq.setParameter('createdAt', LocalDateTime.now(), TemporalType.TIMESTAMP)
            nq.uniqueResult()
    } as Retail
    retail.aggregationResponse = response
    retail

#### HIBERNATE CALL

    Hibernate: CALL SP_retail_find_create_by_response(?, ?)

#### SP

    PROCEDURE SP_retail_find_create_by_response(IN _response BIGINT, IN _created_at DATETIME)
    BEGIN
        DECLARE _id BIGINT;
        
        SELECT id INTO _id FROM retail WHERE aggregation_response_id = _response;
        IF _id IS NULL THEN
            INSERT INTO retail (version, date_created, last_updated, aggregation_response_id, active) 
                        VALUES (0, _created_at, _created_at, _response, TRUE);
            SELECT LAST_INSERT_ID() INTO _id;
        END IF;
        
        SELECT * FROM retail WHERE id = _id LIMIT 1;
    END