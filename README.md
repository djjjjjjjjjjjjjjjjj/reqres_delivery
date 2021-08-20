# delivery user2111111111


##Request.java
```java
@PostPersist
    public void onPostPersist(){

        // requestId
        // "01" : Deposit
        // "02" : Withdraw
        // "03" : Balance
        // "04" : Transaction  
        // "05" : addAccount
        // "06" : deleteAccount
        if( "01".equals( getRequestId() ) || // Deposit
            "02".equals( getRequestId() ) || // Withdraw
            "05".equals( getRequestId() ) || // addAccount
            "06".equals( getRequestId() ) ){ // deleteAccount 
            onlinebank.external.Auth auth = new onlinebank.external.Auth();
            BeanUtils.copyProperties(this, auth);
            auth.setBankRequestId( getId() );
            BankRequestApplication.applicationContext.getBean(onlinebank.external.AuthService.class).requestAuth(auth);
        }
        
        // Balance
        if( "03".equals( getRequestId() ) ){ 
            BalanceRequested balanceRequested = new BalanceRequested();
            BeanUtils.copyProperties(this, balanceRequested);
            balanceRequested.setBankRequestId( getId() );
            balanceRequested.publish();
        }

        // Transaction
        if( "04".equals( getRequestId() ) ){ 
            TransactionRequested transactionRequested = new TransactionRequested();
            BeanUtils.copyProperties(this, transactionRequested);
            transactionRequested.publishAfterCommit();
        }
    }
```






