# UserService
## register
    User user = new User(
            username: command.username,
            password: command.password,
            email: command.email
    )
    ...
    User toValidateExistance = User.findByUsername(user.username)
    user = user.save()
    ...
    Role role = Role.findByAuthority("ROLE_USER")
    UserRole ur = UserRole.create(user, role).save()
    Token tkn = new Token(user: user, tokenType: TokenType.CONFIRMATION).save()
    if (!emailSent) {
        tkn.delete(flush:true, failOnError:true)
        ur.delete(flush:true, failOnError:true)
        user.delete(flush:true, failOnError:true)
    }
## validate
    User user = User.findByUsername(vcc.username)
    ...
    user.apiKey = apiK
    ...
    Token tkn = Token.findByTokenAndUserAndActive(verificationCode, user, true)
    ...
    tkn.with {
        user.enabled = true
        user.save()

        active = false
        token = "0000"
        save()
    }
## sendRecoveryToken
    User user = User.findByUsername(username)
    ...
    Token token = Token.findOrCreateByUserAndTokenType(user, TokenType.RECOVERY)
    token.active = true
    token.token = token.generateOtp()
    token.save(flush: true)
## resetPassword
    User user = User.findByUsername(rpc.username)
    ...
    Token tkn = Token.findByTokenAndUserAndActive(verificationCode, user, true)
    tkn.with {
        user.password = rpc.password
        user.save()

        active = false
        token = "0000"
        save()
    }
## resendTokenConfirmation
    User user = User.findByUsername(username)
    ...
    Token token = Token.findOrCreateByUserAndTokenType(user, TokenType.CONFIRMATION)
    token.active = true
    token.token = token.generateOtp()
    token.save(flush: true)
## validateVerifiedAccount
    User user = User.findByUsernameAndEnabled(username, true)
## updateApiKey
    user.apiKey = generateRequestHash()
    user.save(flush: true)