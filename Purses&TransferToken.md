## How to Create a purse
Purses are places where you can transfer your token to, Each account has its own `Main_Purse` which is an address on the network, you can transfer tokens from your purse to another, using session code

Also your smart contracts can create different purses and control them, they can manage tokens and transfer them to another purse (that belongs to either a account or another smart contract).


### Session Code case
You can access your account's purse using session code, and transfer tokens from your purse to another purse.

Here's how you can do it :

- Transfer from your account to another account or purse (managed by a smart contract) :
    ```Rust
    //this function will be called when you deploy it on network
    #[no_mangle]
    pub extern "C" fn call(){
        //get your own purse to transfer from it
        let own_purse = get_main_purse();
        //get your target accounts hash to transfer to it
        //you could use transfer_from_purse_to_public_key and get target's public key instead
        let target_account : AccountHash = get_named_arg("target");
        //get the amount of casper you want to transfer , the amount is in motes
        //each cspr token is 10^9 motes
        let amount : U512 = get_named_arg("amount");
        //transfer tokens
        system::transfer_from_purse_to_account(own_purse, target_account, amount, Some(1234))
    }
    ```


### Smart-Contract context :
You can create new purses in your contract's context and store it in your contract's sotrage(named keys).

You can access your purse (from NamedKeys) and control it

Here's an example of contract code in which you can build a purse (and save it in named keys), and transfer tokens from it to another purse :

```Rust
#[no_mangle]
pub extern "C" fn create_contract_purse(){ 
    // Creates a purse which can be controlled by the contract
    let purse_addr : URef = system::create_purse();
    // puts the created purse into the storage (named keys) of the contract, so that 
    // it would be possible for contract to access it later
    runtime::put_key("purse_address", purse_addr.into());
}

#[no_mangle]
pub extern "C" fn get_balance(){
    // get the purse uref from storage (named keys)
    let purse_ref = get_key("purse_address").unwrap_or_revert().into_uref().unwrap_or_revert();
    
    // get balance of the purse with system::get_purse_balance
    let balance = get_purse_balance(purse_ref).unwrap_or_revert();
    // To return that value to the caller (ex. another contract), you should wrap it in a CLValue
    let value = CLValue::from_t(balance).unwrap_or_revert();
    // Return the value to the caller (a.k.a another smart contract)
    runtime::ret(value);
}


#[no_mangle]
pub extern "C" fn foo(){
    // get the purse refrence from named keys
    let purse_addr : URef = get_key("purse_address").unwrap_or_revert().into_uref().unwrap_or_revert();
    // read target's account hash from session arguments
    let target_account : AccountHash = get_named_arg("target");
    // transfer 1000000 motes from your purse to target's purse
    system::transfer_from_purse_to_account(purse_addr, target_account, U512::from_dec_str("1000000").unwrap_or_default(), Some(1234u64));
}
```

<footer>
<p style = "float:left; width : 20%;">
<a href = "https://github.com/k3rn3lpanicc/CasperNetwork-by-example">Previous</a>
</p>
<p style = "float:right; width : 5%;">
<a href = "">Next</a>
</p>
