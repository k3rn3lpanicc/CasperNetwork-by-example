## EntryPoints
EntryPoints are methods, which can be called by another method(another contract calls this entrypoint) or by a user.

Each EntryPoint has :
- `name` : name **must** be exactly equal to the method's name 
- `args` : args is a `Parameters` object, It is a `Vec<Parameter>`. Each `Parameter` has a `name:&str` and a `CLType`
- `ret` : return type, it will be specified by a `CLType` object
- `access` : It's either `EntryPointAccess::Public` which indicates this entrypoint can be called by anyone on network, or `EntryPointAccess::Groups(...)` which indicates that this entrypoint can be called only by a special group of users or contracts.
- `entry_point_type` : It's either `Contract` type or `Session` type. Contract entrypoints will have access to the storage of the SmartContract, but on the other hand, session entrypoints will be executed on the User-Account's context and has access to Named-Keys of the user (not the contract).


### Usage of EntryPoints

EntryPoints can be used to define a Smart-Contract's methods that can be used on the network, so we define them in order to use them as methods of contracts.

Below is an example of making a contract which has 2 entrypoints. Please note that `call` function is not an entrypoint that you can call directly, Instead, it will be executed the moment you deploy your compiled WASM file, so `call` entrypoint will be executed once, and as you can guess, It's `entry_point_type` is `Session` (it will be executed in your account's context).

```Rust
#[no_mangle]
pub extern "C" fn foo1(){
    let arg1 : String = get_named_arg("arg1");
    let arg2 : u32 = get_named_arg("arg2");
    // some code here :P

    //return the 10 as u32 value to the caller
    runtime::ret(CLValue::from_t(10u32).unwrap_or_revert());
}
#[no_mangle]
pub extern "C" fn foo2(){
    let my_purse = get_main_purse();
    let account_hash : AccountHash = get_named_arg("target");
    transfer_from_purse_to_account(my_purse, account_hash, U512::from_str("1000000000"), Some(1234));
}

#[no_mangle]
pub extern "C" fn call(){
    let entry_points = EntryPoints::new();  
    let mut param = Parameters::new();
    entry_points.add_entry_point(EntryPoint::new("foo", vec![
        Parameter::new("arg1", CLType::String),
        Parameter::new("arg2", CLType::U32)
    ], CLType::U32, EntryPointAccess::Public, EntryPointType::Contract));

    let mut param2 = Parameters::new();
    entry_points.add_entry_point(EntryPoint::new("foo2", vec![
        Parameter::new("target", CLType::Key),
    ], CLType::U32, EntryPointAccess::Public, EntryPointType::Session));

    
    let (contract_hash , contract_version) = storage::new_contract(entry_points, NamedKey::default(), Some("lol".to_string()), Some("lol2".to_string()));
    runtime::put_key("my_contract", contract_hash.into());
}

```
