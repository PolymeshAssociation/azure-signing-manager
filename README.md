[![js-semistandard-style](https://img.shields.io/badge/code%20style-semistandard-brightgreen.svg?style=flat-square)](https://github.com/standard/semistandard)
[![semantic-release](https://img.shields.io/badge/%20%20%F0%9F%93%A6%F0%9F%9A%80-semantic--release-e10079.svg)](https://github.com/semantic-release/semantic-release)

# Azure Signing Manager

The AzureSigningManager implements the Polymesh SDK signing manager interface. This allows Polymesh transactions to be signed with keys in an Microsoft Azure [key vault](https://azure.microsoft.com/en-us/products/key-vault). The keys must be "EC"
type and use curve "P-256K". The signing manager will ignore any other type.

## Usage

```typescript
import { AzureSigningManager } from '@polymeshassociation/azure-signing-manager';
import { Polymesh } from '@polymeshassociation/polymesh-sdk';

// defaults to constructing `new DefaultAzureCredential()` for credential
const signingManager = new AzureSigningManager({
  keyVaultUrl: 'https://somekeyvault.vault.azure.net/',
});

const polymesh = await Polymesh.connect({
  nodeUrl,
  signingManager,
});

const newKey = await signingManager.createKey('myKey') // keys can be created in the Azure UI or CLI as well
console.log('created key with address: ', newKey.address) // address is the primary way of specifying public keys on Polymesh
```

## Authorization

To authorize access to the key vault a `DefaultAzureCredential` will be created. By default it searches for a credential in this order:

1. `EnvironmentCredential`
1. `WorkloadIdentityCredential`
1. `ManagedIdentityCredential`
1. `AzureCliCredential`
1. `AzurePowerShellCredential`
1. `AzureDeveloperCliCredential`

More details about authorization can be found on the [Azure Docs](https://learn.microsoft.com/en-us/javascript/api/@azure/identity/defaultazurecredential?view=azure-node-latest#@azure-identity-defaultazurecredential-constructor). Optionally, a credential can be passed instead.

The identity will need permission to read and to sign with the keys. In order for the createKey function to work then create permission will be required as well. At least one of the roles "Key Vault Crypto User" or "Key Vault Crypto Officer" should be assigned. There is more info in the [official guide](https://learn.microsoft.com/en-us/azure/key-vault/general/rbac-guide)

## Pricing Note (for HSM keys)

Storing many HSM keys can be pricy
```
First 250 keys	         $5 per key per month
From  251 – 1,500 keys	 $2.50 per key per month
From  1,501 – 4,000 keys $0.90 per key per month
4,001+ keys	             $0.40 per key per month
+ $0.15/10,000 transactions
```
> Only actively used HSM protected keys (used in prior 30-day period)

Where as software protected keys are charged only the per transaction fee of $0.15/10,000.

See the [pricing page](https://azure.microsoft.com/en-us/pricing/details/key-vault/) for details.

If you need large amounts of keys for your use case please reach out via [support](https://polymesh.network/contact-us) to find the best key storage solution for your use case.