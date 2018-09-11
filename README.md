# What is this

This is the fitchain registry on ethereum.
The smart contracts part of the fitchain platform are

- fitchainRegistrar: registry of actors (data owner, data scientist, validator)
- fitchainRegistry:  registry of models and model challenges


## Testing

1. Start testrpc (or ganache-cli)

``` $ ganache-cli ```


2. Set sender address in ```truffle.js``` with one account created in ```ganache```


```javascript
module.exports = {
  networks: {
    development: {
      host: "localhost",
      port: 8545,
      from: "ganache sender address",  
      gas: 5000000,
      network_id: "*" // Match any network id
    }
  }
};
```

3. Launch truffle

` $ truffle test `

This should return

```
Using network 'development'.

Compiling ./contracts/fitchainLib.sol...
Compiling ./contracts/fitchainRegistrar.sol...
Compiling ./contracts/fitchainRegistry.sol...

  Contract: fitchainRegistrar
    ✓ adding registrant to the registrar (402ms)
    ✓ deactivate registrant (84ms)
    ✓ check if registrant is active

  Contract: fitchainRegistry
    ✓ adding models to the registry  (180ms)
    ✓ setting model as valid/invalid
    ✓ getting models from the registry  (39ms)
    ✓ adding challenges to existing model  (370ms)
    ✓ setting challenge as active/inactive (49ms)
    ✓ getting all model challenges (205ms)
    ✓ getting some challenges (47ms)
    ✓ deleting one model from the registry  (312ms)


  11 passing (2s)


```

## Contract usage documentation

## Contract: fitchainRegistry
### Events

* ```EntityCreated(address owner,
    bytes32 entityType,
    bytes32 entityId,
    bytes32 ipfs) ```

* ```EntityArchived(address owner, bytes32 entityType, bytes32 entityId);```

* ```LogCreated(bytes32 project, bytes32 workspace, bytes32 job, uint timestamp, bytes32 channel, string payload);```

* ```MetricCreated(address user, bytes32 project, bytes32 workspace, bytes32 job, uint timestamp, bytes32 key, bytes32 value);```

* ```WorkspaceCreated(uint timestamp, address user, bytes32 project);```


Create model without duplicates

* <bytes> model_id name of the thing to create  
* <bytes32> ipfs address where thing is stored
* <uint> stored_as how to store thing 0:str, 1:bytes, 2:pyobj
* @return bool - success or fail


**Create Model**
----
  Create model (without duplicates). Returns `true` on success or `false` on failure.

* **Method:**

  fitchainRegistry:createModel

*  **Params**

   **Required:**
   `model_id=[bytes32]`
   `ipfs_address=[bytes32]`
   `stores_as=[uint]`
   `bounty=[uint]`
   `current_error=[uint]`
   `target_error=uint`

* **Data Params**

  None

* **Success Response:**

  * **Code:** `true`
    **Content:** `ipfs_address=[bytes32]`

* **Error Response:**

  {

    **ErrorCode:** 1 MODEL ALREADY EXISTS

    **Content:** `{ error : "Model already exists" }`

  }

* **Events:**
```javascript
Created(ipfs_address, msg.sender);
```

* **Sample Call:**

  ```javascript
  createModel('model_4', '0x2345', '0', '100', '100', '10');
  ```


**Create Model Challenge**
----
  Create challenge for an existing model. Returns the unique id of the challenge created, or 0x0 on failure.

* **Method:**

  fitchainRegistry:createChallenge

*  **Params**

   **Required:**

   `model_id=[bytes32]`
   `ipfs_address=[bytes32]`

* **Data Params**

  None

* **Success Response:**

  * **Code:** 200 <br />
    **Content:** `ipfs_address=[bytes32]`

* **Error Response:**

  * **ErrorCode:** 2 MODEL NOT FOUND <br />
    **Content:** `{ error : "Model doesn't exist" }`

* **Events:**
```javascript
Created(ipfs_address, msg.sender);
```


* **Sample Call:**

  ```javascript
    createChallenge('0xabcd', '0x123456');
  ```



**Get number of models**
  ----
Returns the number of unique models in the registry.

* **Method:**

    fitchainRegistry:getNumberOfModels

*  **Params**

    None

* **Success Response:**

    **Content:** `num_models=[uint]`

* **Sample Call:**

  ```javascript
    getNumberOfModels();
  ```



**Get number of challenges**
    ----
  Returns the number of challenges of an existing model.

* **Method:**

      fitchainRegistry:getNumberOfChallenges

*  **Params**

   **Required:**

       `model_id=[bytes32]`

  * **Data Params**

      None

* **Success Response:**

  * **Code:** 200 <br />
    **Content:** `num_challenges=[uint]`

* **Error Response:**

  * **ErrorCode:** 2 MODEL NOT FOUND <br />
    **Content:** `{ error : "Model doesn't exist" }`

* **Event:**

  None


* **Sample Call:**
    ```javascript
    getNumberOfChallenges('0xabcd')
    ```


**Get model**
    ----
Get all fields of model with `model_id`.

* **Method:**

    fitchainRegistry:getModel

*  **Params**

     **Required:**

     `model_id=[bytes32]`

  * **Data Params**

      None

* **Success Response:**

    * **Code:** 200 <br />
      **Content:**

          ownerAddress=[address],
          ipfsAddress=[bytes32],
          storedAs=[uint],
          bounty=[uint],
          currentError=[uint],
          targetError=[uint],
          isValid=[bool]`

* **Error Response:**

  * **ErrorCode:** 2 MODEL NOT FOUND

    **Content:** `{ error : "Model doesn't exist" }`


* **Events:**

  None


* **Sample Call:**

    ```Javascript
        const model_1 = getModel('0xabcd');
    ```


**Get Challenge**
        ----
Get all fields of challenge with `challenge_hash`.

* **Method:**

    fitchainRegistry:getChallenge

*  **Params**

   **Required:**

     `challenge_hash=[bytes32]`

   * **Data Params**

          None

* **Success Response:**

    * **Code:** 200 <br />
      **Content:**
          model_identity=[bytes32],
          verifier_address=[address],
          ipfs_address=[bytes32],
          error_metric=[uint],
          is_active=[bool],
          validator_address=[bytes32]

* **Error Response:**

  * **ErrorCode:** 2 MODEL NOT FOUND <br />
    **Content:** `{ error : "Challenge doesn't exist" }`

* **Event:**

      None

* **Sample Call:**

  ```javascript
        const challenge_1 = getChallenge('0x123456');
    ```




**Get model challenges**
                ----
  Get all challenge ids of the model passed as parameter.

  * **Method:**

    fitchainRegistry:getModelChallenges

  *  **Params**

     **Required:**

     `model_id=[bytes32]`

     * **Data Params**

        None

  * **Success Response:**

  * **Code:** 200 <br />
    **Content:**

        list of challenge hashes =[bytes32[]],


  * **Error Response:**

    * **ErrorCode:** 2 MODEL NOT FOUND <br />
      **Content:** `{ error : "Model doesn't exist" }`

* **Event:**

    None

  * **Sample Call:**

  ```javascript
    var challenges_1 = getModelChallenges('0xabcd');
  ```





**Delete model**
  ----
Delete model with `model_id` from registry.

**Warning** All challenges associated to `model_id` will be deleted too.


* **Method:**

      fitchainRegistry:deleteModel

*  **Params**

   **Required:**

     `model_id=[bytes32]`

     * **Data Params**

        None

* **Success Response:**

  * **Content:**

    success=[bool],

  * **Error Response:**

    * **ErrorCode:** 2 MODEL NOT FOUND <br />
      **Content:** `{ error : "Model doesn't exist" }`

    * **ErrorCode:** 3 NOT AUTHORIZED <br />
      **Content:** `{ error : "Not authorized error" }`

* **Event:**

  ```javascript
    Deleted(model_id);  
    ```

* **Sample Call:**

    ```javascript
      deleteModel('0xabcd');
    ```





**Delete challenge**
      ----
  Delete challenge with `challenge_hash` from registry

* **Method:**

    fitchainRegistry:deleteChallenge

*  **Params**

   **Required:**

   `model_id=[bytes32]`

   * **Data Params**

      None

* **Success Response:**

  * **Code:** 200 <br />
    **Content:**

    list of challenge hashes =[bytes32[]],


* **Error Response:**

    * **ErrorCode:** 2 CHALLENGE NOT FOUND <br />
      **Content:** `{ error : "Model doesn't exist" }`

    * **ErrorCode:** 3 NOT AUTHORIZED <br />
      **Content:** `{ error : "Not authorized error" }`

* **Event:**

```javascript
  Deleted(challenge_id);  
  ```

* **Sample Call:**

  ```javascript
  deleteChallenge('0x123456000000000000000000000000000000000');
  ```



**Set model valid/invalid**
        ----
    Set the validity flag for model with `model_id`
* **Method:**

      fitchainRegistry:setModelValid

*  **Params**

   **Required:**

     `model_id=[bytes32]`
     `is_valid=[bool]`

   * **Data Params**

        None

* **Success Response:**

    * **Code:** 200 <br />

      **Content:**

        success=[bool],

* **Error Response:**

    * **ErrorCode:** 2 MODEL NOT FOUND <br />
        **Content:** `{ error : "Model doesn't exist" }`

    * **ErrorCode:** 3 NOT AUTHORIZED <br />
        **Content:** `{ error : "Not authorized error" }`

* **Event:**
  ```javascript
    Updated(model_id, ownerAddress, isValid);  
  ```


* **Sample Call:**

    ```javascript
    const ret = setModelValid('0xabcd', false);
    ```



**Set challenge active/inactive**
        ----
  Set challenge with `challenge_id` as active/inactive

* **Method:**
        fitchainRegistry:setChallengeValid

*  **Params**

   **Required:**

   `challenge_id=[bytes32]`
   `is_active=[bool]`

   * **Data Params**

      None

* **Success Response:**

  * **Code:** 200 <br />

    **Content:**
          success=[bool],

* **Error Response:**

  * **ErrorCode:** 2 CHALLENGE NOT FOUND

    **Content:** `{ error : "Challenge doesn't exist" }`

  * **ErrorCode:** 3 NOT AUTHORIZED

    **Content:** `{ error : "Not authorized error" }`


* **Event:**
```javascript
Updated(challenge_id, verifierAddress, isActive);  
```

* **Sample Call:**

  ```javascript
  const ret = setChallengeActive('0x123456', false);
  ```