# 3box-js data structure specification
The implementation of this can be found at [3box-js](https://github.com/uport-project/3box-js)

## Data structure for users data stores
Each user has its own separate ipfs data structure in 3box. This data structure consists of thee parts; the public profile, the private data store, and a root object that points to the latest versions of these. As long as the user has access to the hash of the root object it can retrieve the entire data store from the ipfs network.

### Root object
The root object is an IPLD formated ipfs object that contains a link to the latest hash of the public profile and a link to the latest hash of the private data store. It have the following structure:

```js
{
  profile: {"/" : "zdpuAufy3hawb25akerURtR81y7D4BKfdxfqbpYZ7cJGjwgFW"},
  datastore: {"/" : "zdpuAufy3hawb25akerURtR81y7D4BKfdxfqbpYZ7cJGjwgDS"}
}
```

### Public profile
The public profile is an IPLD formated ipfs object that contains the public information about the user such as name and picture. We use the Profile scheme from <http://schema.org/>, with extensions by [Blockstack](https://github.com/blockstack/blockstack.js/tree/master/src/profiles), and using IPLD links. We start with just the items `name` and `image`. Note that the `image` field is an array.

```js
{
  "@context": "http://schema.org/",
  "@type": "Person",
  "name" : "Christian",
  "image" : [{"@type": "ImageObject", "contentUrl": {"/" : "QmXXXX"}}].
}
```

### Private data store
The private data store is an orbit-db KV-store with additional encryption. The encryption scheme for adding a key-value entry would work as follows:

* Generate a random salt and store it encrypted (see below) under the `3BOX_SALT` plain text key.
* Compute the `key` by taking `h(PLAIN_KEY | salt)`
* Generate a random nonce `n`
* Add padding to `PLAIN_VALUE` so that its length is a multiple of `24`
* Compute the ciphertext: `nacl.secretbox(PLAIN_VALUE, n, secretKey)`
* Encode `value` as `{ nonce: Base64(n), ciphertext: Base64(ciphertext) }`

We can now store `key` and `value` in the orbit-db KV-store using the `put` method.

Optionally we can add an index of encrypted keys to the db.
