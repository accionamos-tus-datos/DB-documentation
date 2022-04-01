# JwtTokenService
## getJwtToken
    List tkns = UnnaxToken.findAll()
## getNewToken
    if(!tkn) {
        tkn = new UnnaxToken()
    }
    ...
    tkn.with {
        accessToken = json.access
        refreshToken = json.refresh
        createdToken = LocalDateTime.now()
        refreshedToken = LocalDateTime.now()
    }
    tkn.save(flush: true)
## refreshToken
    UnnaxToken tkn
    tkn.accessToken = json.access
    tkn.refreshedToken = LocalDateTime.now()
    tkn.save(flush: true)