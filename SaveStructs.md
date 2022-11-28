## How to save Rust structs in storage and retrieve them

Suppose you're developing a Smart-Contract on casper network and you're defining your structs and data-structures for your development, You can save your struct on blockchain (main storage or as a dictionary's value) and get it back easily, all you have to do is implement `ToBytes`, `FromBytes` and `CLTyped` traits for your struct.

Below's an example of implementation of these traits : 

```Rust
pub struct NFTHolder {
    pub remaining_amount : u64,
    pub amount : u64,
    pub token_id : u64
}
//Implement ToBytes, FromBytes and CLTyped for NFTHolder to be able to store it in the contract's storage and retrieve it
impl ToBytes for NFTHolder{
    fn to_bytes(&self) -> Result<Vec<u8>, casper_types::bytesrepr::Error> {
        let mut result = Vec::new();
        result.append(&mut self.remaining_amount.to_bytes()?);
        result.append(&mut self.amount.to_bytes()?);
        result.append(&mut self.token_id.to_bytes()?);
        Ok(result)
    }
    fn into_bytes(self) -> Result<Vec<u8>, casper_types::bytesrepr::Error>
    where
        Self: Sized,
    {
        self.to_bytes()
    }
    fn serialized_length(&self) -> usize{
        self.remaining_amount.serialized_length() + self.amount.serialized_length() + self.token_id.serialized_length()
    }
}
impl FromBytes for NFTHolder{
    fn from_bytes(bytes: &[u8]) -> Result<(Self, &[u8]), casper_types::bytesrepr::Error> {
        let (remaining_amount, rem) = FromBytes::from_bytes(bytes)?;
        let (amount, rem) = FromBytes::from_bytes(rem)?;
        let (token_id, rem) = FromBytes::from_bytes(rem)?;
        Ok((NFTHolder{remaining_amount, amount, token_id}, rem))
    }
    fn from_vec(bytes: Vec<u8>) -> Result<(Self, Vec<u8>), casper_types::bytesrepr::Error> {
        Self::from_bytes(bytes.as_slice()).map(|(x, remainder)| (x, Vec::from(remainder)))
    }
}

//CLTyped trait is used by casper to determine the type of your data (to save it on storage, for your structs, you should use ByteArray with size of your struct (because we're serializing structs to byte arrays))
impl CLTyped for NFTHolder{
    fn cl_type() -> casper_types::CLType {
        casper_types::CLType::ByteArray(24u32)
    }
}

```

By implementing these traits for your struct, you should be able to easily treat them as other primitive types that you could save and retrieve them like so :

```Rust
#[no_mangle]
pub extern "C" fn call(){
    let holder = ndpc_types::NFTHolder::new(10u64 , 10u64, 0u64);
    let dict = storage::new_dictionary("dict_name").unwrap_or_revert();
    storage::dictionary_put(dict, "dict_key", holder);
    let holder_retreived : NFTHolder = storage::dictionary_get(dict, "dict_key").unwrap_or_revert();
}
```
