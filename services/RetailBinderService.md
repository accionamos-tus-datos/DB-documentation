# RetailBinderService
## marshallJsonToDomains
    Retail retail = Retail.findOrCreateByAggregationResponse(aResponse)
    if(!retail.id) {
        retail.save(flush: true)
    }
    retails = retail?.save(flush: true) 
    retail != null
## saveCustomer
    CustomerRetail customer = CustomerRetail.findOrCreateByNameAndClientIdAndRetail(c['names'], c['client_id'], _retail)
    customer.with {
        unnaxId = c['_id']
        birthDate = c['birth_date']
        gender = (Integer) c['gender']
        active = true
    }
    CustomerRetail.findAllByClientIdNotInListAndRetailAndActive(clientsIds, _retail, true).each {
        it.active = false
        customer.retail = _ retail
        customer = customer.save()
    }
## saveContacts
    List<CustomerContactRetail> _contacts = CustomerContactRetail.findAllByCustomerAndType(_customer, _type) //_customer.contacts?.findAll {it.type == _type}?.asList() ?: []
    
    contacts.each { String _data ->
        CustomerContactRetail ccr = _contacts.find {it.data == _data} ?: new CustomerContactRetail()
        if (!ccr.id) {
            ccr.data = _data
            ccr.type = _type
        }
        ccr.active = true
        ccr.customer = _customer
        ccr = ccr.save()
    }
    _contacts.findAll {!(it.data in contacts) && it.active}.each {
        it.active = false
        ccr.customer = _customer
        ccr = ccr.save()
    }

## saveShopping
    List<ShoppingRetail> byPurchaseDate = ShoppingRetail.findAllByRetailAndPurchaseDate(_retail, date) //currents?.findAll { it.purchaseDate == date } ?: []
    List<Map> input
    
        ... // creacion o actualización
        input.each { i ->

            ShoppingRetail s = byPurchaseDate.find { it.lookupId == i.lookup_id && it.customerRetail.clientId == i.customer_id }
            if (s != null) {
                s.unnaxId = i['_id']
                s.paymentMethod = i.payment_method
                s.invoice = i.invoice
                s.shippingAddress = i.shipping_address.join(" ")
                s.totalAmount = i.total_amount
                s.active = true
            } else {
                s = new ShoppingRetail().with {
                    unnaxId = source['_id']
                    customerRetail = customer
                    lookupId = source.lookup_id
                    purchaseDate = source.purchase_date
                    paymentMethod = source.payment_method
                    invoice = source.invoice
                    shippingAddress = source.shipping_address.join(" ")
                    totalAmount = source.total_amount
                    active = true
                    retail = _retail
                    save()
                }
            }

        }
        ... // creacion y desactivar
        input.eachWithIndex{ Map entry, int i ->
            ShoppingRetail s = byPurchaseDate[i].with {
                unnaxId = source['_id']
                customerRetail = customer
                lookupId = source.lookup_id
                purchaseDate = source.purchase_date
                paymentMethod = source.payment_method
                invoice = source.invoice
                shippingAddress = source.shipping_address.join(" ")
                totalAmount = source.total_amount
                active = true
                retail = _retail
                save()
            }
        }
        for (int i = input.size(); i < byPurchaseDate.size(); i++) {
            byPurchaseDate[i].active = false
            byPurchaseDate[i].save(flush: true)
            s.retail = _retail
            s = s.save()
        }

## saveShoppingCredits
    ShoppingCreditRetail.findAllByRetailAndActive(_retail, true).each {
        it.active = false
        it.save()
    }
    inputList.each { newCredit ->
        ShoppingCreditRetail credit = ShoppingCreditRetail.findAllByRetailAndLookupId(_retail, newCredit.lookup_id) //_retail.shoppingCredits.find {it.lookupId == newCredit.lookup_id}

        (credit ?: new ShoppingCreditRetail()).with {
            unnaxId = source['_id']
            customerRetail = customer
            lookupId = source.lookup_id
            currentBalance = source.current_balance
            currentDebt = source.current_debt
            creditAvailable = source.credit_availabl
            minPayment = source.min_payment
            noInterestPayment = source.no_interest_payment
            dueDate = source.payment_due_date
            walletBalance = source.wallet_balance
            active = true
            retail = _retail
            save()
        }
    }

## saveShoppingCreditBillings
    List<ShoppingCreditBillingRetail> billings = ShoppingCreditBillingRetail.findAllByRetailAndDueDate(_retail, _dueDate)//_retail.shoppingCreditBillings?.findAll { it.dueDate == sc.key }?.asList() ?: []
    List<Map> items //Entry list
    
        ... //creacion o actualizacion
        billings.eachWithIndex { ShoppingCreditBillingRetail entry, int i ->
            Map source = items[i]
            entry.with {
                unnaxId = source['_id']
                customerRetail = customer
                lookupId = source.lookup_id
                period = source.period
                expense = source.expense
                payments = source.paytment
                dueDate = source.due_date
                active = true
                retail = _retail
                save()
            }
        }
        for (int i = billings.size(); i < items.size(); i++) {
            Map source = items[i]
            new ShoppingCreditBillingRetail().with {
                unnaxId = source['_id']
                customerRetail = customer
                lookupId = source.lookup_id
                period = source.period
                expense = source.expense
                payments = source.paytment
                dueDate = source.due_date
                active = true
                retail = _retail
                save()
            }
        }
    
        ... //Actualización y desactivación
        items.eachWithIndex { Map entry, int i ->
            billings[i].wtih {
                unnaxId = source['_id']
                customerRetail = customer
                lookupId = source.lookup_id
                period = source.period
                expense = source.expense
                payments = source.paytment
                dueDate = source.due_date
                active = true
                retail = _retail
                save()
            }
        }
        for (int i = items.size(); i < billings.size(); i++) {
            billings[i].active = false
            billings[i].retail = _retail
            billings[i].save(flush: true)
        }
## saveItemsShopping
    List items //Entry list
    List<ItemShoppingRetail> shoppingItems = ItemShoppingRetail.findAllByShopping(_shopping) //shopping.items?.toList() ?: []
    
        ... // Creacion y actualizacion
        shoppingItems.eachWithIndex { ItemShoppingRetail entry, int i ->
            source = items[i]
            entry.with {
                unnaxId = source['_id']
                name = source.names
                amount = source.amount
                count = source.count as Integer
                discount = source.discount
                totalAmount = source.total_amount
                active = true
                shopping = _shopping
                save()
            }
        }
        for (int i = shoppingItems.size(); i < items.size(); i++) {
            source = items[i]
            new ItemShoppingRetail().with {
                unnaxId = source['_id']
                name = source.names
                amount = source.amount
                count = source.count as Integer
                discount = source.discount
                totalAmount = source.total_amount
                active = true
                shopping = _shopping
                save()
            }
        }
        ...//Actualización y desactivacion
        items.eachWithIndex { Map source, int i ->
            shoppingItems[i].with {
                unnaxId = source['_id']
                name = source.names
                amount = source.amount
                count = source.count as Integer
                discount = source.discount
                totalAmount = source.total_amount
                active = true
                shopping = _shopping
                save()
            }
        }
        for (int i = items.size(); i < shoppingItems.size(); i++) {
            shoppingItems[i].active = false
            shoppingItems[i].shopping = _shopping
            shoppingItems[i].save()
        }
    }