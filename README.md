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


##Auth.java

```java
package onlinebank;

@Entity
@Table(name="Auth_table")
public class Auth {

    @PrePersist
    public void onPrePersist(){

        String userId = "1@sk.com";
        String userName = "sam";
        String userPassword = "1234";
        boolean authResult = false ;

        if( userId.equals( getUserId() ) && userName.equals( getUserName() ) && userPassword.equals( getUserPassword() ) ){
            authResult = true ;
        }

        if( authResult == false ){
            AuthCancelled authCancelled = new AuthCancelled();
            BeanUtils.copyProperties(this, authCancelled);
	    // 실패 이벤트 카프카 송출
            authCancelled.publish();

        }else{
            AuthCertified authCertified = new AuthCertified();
            BeanUtils.copyProperties(this, authCertified);

            TransactionSynchronizationManager.registerSynchronization(new TransactionSynchronizationAdapter() {
                @Override
                public void beforeCommit(boolean readOnly) {
		    // 성공 이벤트 카프카 송출
                    authCertified.publish();
                }
            });
        }
    }

```

