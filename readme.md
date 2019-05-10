<h1 align='center'>
  <a href='https://rinkeby.aragon.org/#/0xe520428C232F6Da6f694b121181f907931fD2211'>1Hive DAO 🐝</a>
</h1>

<br>

# 1Hive Deployment Log

1Hive is an Aragon organization, currently deployed to rinkeby.

The following log describes the deployment process using the Aragon CLI. If your walking through this as a guide be sure to change the variables to correspond to addresses for your deployment.

The docs for the CLI used in this deployment can be found [here](https://hack.aragon.org/docs/cli-dao-commands).

You will also need to [have IPFS running](https://hack.aragon.org/docs/cli-ipfs-commands) in a separate window.

---

### 1. Deploy a fresh dao

`dao --environment aragon:rinkeby new`

This will output the deployed token address. For legibility of subsequent commands will set a bash environment variable for the address:

`onehive=0xe520428C232F6Da6f694b121181f907931fD2211`

We should also set keep track of EOA used for Deployment
`deployer=0x625236038836CecC532664915BD0399647E7826b`

---

### 2. Deploy the organizations membership token `Bee`

`dao --environment aragon:rinkeby token new "Bee" "Bee" 0`

This will output the deployed token address. For legibility of subsequent commands will set a bash environment variable for the address:

`token_bee=0xfaE3B25ec796cF099fE1e7ba21e6d99297640829`

---

### 3. Deploy a voting app instance tied to the `Bee` this voting app will serve as the root authority in the organization.

We are initializing the voting app with a 50 percent support requirement, a 5 percent approval quorum, and one week vote duration.

`dao --environment aragon:rinkeby install $onehive voting --app-init-args $token_bee 500000000000000000 50000000000000000 604800`

To get the address of an installed app we need to check permissionless apps:

`dao --environment aragon:rinkeby apps $onehive --all`

And we can set this address to an environment variable:

`voting_bee=0x7d8913cbB31c8D7E90Ed918B55F02DB329622bEd`

For this to be useful we also need to create the initial permission, we can do this by self-assigning the voting address as manager, and granting the create votes rule to EOA used for deployment.

`dao --environment aragon:rinkeby acl create $onehive $voting_bee CREATE_VOTES_ROLE $deployer $voting_bee`

---

### 4. Deploy a Token Manager instance to manage the `Bee` token to our dao

`dao --environment aragon:rinkeby install $onehive token-manager --app-init none`

Then we can set a environment variable for the address:

`manager_bee=0x3C16dC46b84a6647f8375235cA88DAd2C27Edb8B`

We can then set the token manager as the token controller for `Bee`

`dao --environment aragon:rinkeby token change-controller $token_bee $manager_bee`

And we can set the permissions such so that bee voting is required to Issue, Mint, Assign, and Burn.

`dao --environment aragon:rinkeby acl create $onehive $manager_bee MINT_ROLE $deployer $voting_bee`

`dao --environment aragon:rinkeby acl create $onehive $manager_bee ISSUE_ROLE $voting_bee $voting_bee`

`dao --environment aragon:rinkeby acl create $onehive $manager_bee ASSIGN_ROLE $voting_bee $voting_bee`

`dao --environment aragon:rinkeby acl create $onehive $manager_bee BURN_ROLE $voting_bee $voting_bee`

Finally we need to initialize the token manager, setting the token to non-transferrable and limiting balances to 1

`dao --environment aragon:rinkeby exec $onehive $manager_bee initialize $token_bee false 1`

---

### 5. Deploy the organization's contribution token `Honey`

`dao --environment aragon:rinkeby token new "Honey" "Honey"`

Set a variable for the `Honey` token address:

`token_honey=0x4a7683282053FF1e2381Cc1663fb04FbBCD18350`

---

### 6. Install a token manager to manage `Honey`

`dao --environment aragon:rinkeby install $onehive token-manager --app-init none`

Get the new token managers proxy address and set it to an env variable
`dao --environment aragon:rinkeby apps $onehive --all`

`manager_honey=0xDA552be756AEb99Df8d7dED3d853e1d57eFa2442`

Now we can change the controller of the `Honey` token to the new token manager

`dao --environment aragon:rinkeby token change-controller $token_honey $manager_honey`

And we create permissions granting authority to the Bee voting app instance

`dao --environment aragon:rinkeby acl create $onehive $manager_honey MINT_ROLE $voting_bee $voting_bee`

`dao --environment aragon:rinkeby acl create $onehive $manager_honey ISSUE_ROLE $voting_bee $voting_bee`

`dao --environment aragon:rinkeby acl create $onehive $manager_honey ASSIGN_ROLE $voting_bee $voting_bee`

`dao --environment aragon:rinkeby acl create $onehive $manager_honey BURN_ROLE $voting_bee $voting_bee`

And we can initialize the `Honey` token as transferrable without a balance limiting

`dao --environment aragon:rinkeby exec $onehive $manager_honey initialize $token_honey true 0`

---

### 7. Deploy a second voting app instance tied to `Honey`. This voting app will have authority of finances.

We are initializing the voting app with a 50 percent support requirement, a 5 percent approval quorum, and one week vote duration.

`dao --environment aragon:rinkeby install $onehive voting --app-init-args $token_honey 500000000000000000 50000000000000000 604800`

To get the address of an installed app we need to check permissionless apps:

`dao --environment aragon:rinkeby apps $onehive --all`

And we can set this address to an environment variable:

`voting_honey=0xbF76027f3048487be00416b8d480A488F5FF2Bfb`

For this to be useful we also need to create the initial permission, we can do this by self-assigning the voting address as manager, and granting the create votes rule to EOA used for deployment.

`dao --environment aragon:rinkeby acl create $onehive $voting_honey CREATE_VOTES_ROLE $manager_bee $voting_bee`

---

### 8. Install a vault and the finance app

`dao --environment aragon:rinkeby install $onehive vault`

Set the address to a variable:

`vault_finance=0x4dDd23610430635b809384B3bD6f54c373A1dDd0`

Then we install finance:

`dao --environment aragon:rinkeby install $onehive finance --app-init-args $vault_finance 2592000`

and set the address to a variable:

`finance=0x1820904E994518332fa6c54eB70F28FCBFF2c45c`

Grant permission for finance to transfer assets from the vault:

`dao --environment aragon:rinkeby acl create $onehive $vault_finance TRANSFER_ROLE $finance $voting_honey`

Grant the `Honey` vote permissions on finance app:

`dao --environment aragon:rinkeby acl create $onehive $finance CREATE_PAYMENTS_ROLE $voting_honey $voting_bee`

`dao --environment aragon:rinkeby acl create $onehive $finance EXECUTE_PAYMENTS_ROLE $voting_honey $voting_bee`

`dao --environment aragon:rinkeby acl create $onehive $finance MANAGE_PAYMENTS_ROLE $voting_honey $voting_bee`

---

### 9. Install Projects and Allocations + dependencies

We want the projects and allocations apps to be connected to a different vault than finance so we need to add a new vault.

`dao --environment aragon:rinkeby install $onehive vault`

And we can set the address to a variable

`vault_allocations=0x4D90a7062711edB307aEdBd7A408ca317c32b4d7`

Then we can install the address book app:

`dao --environment aragon:rinkeby install $onehive tps-address-book.open.aragonpm.eth`

Set a variable for address book:

`address_book=0xCAD8C55c4129F8f7ecC623a3FE5b6E705DCF9F2b`

Then we can set up the dot voting app linked to `Honey`:

`dao --environment aragon:rinkeby install $onehive tps-dot-voting.open.aragonpm.eth --app-init-args $address_book $token_honey 500000000000000000 0 604800`

Set a variable for dot voting
`dot_voting=0x6d39A4151ebcaD63dde880518e110dE0942753a2`

Before we install the projects app we can set a variable for the standard bounties contract on rinkeby:

`bounties=0xcac024cb2ad5f22c3e92053b95c89f69442952d8`

`dao --environment aragon:rinkeby install $onehive tps-projects.open.aragonpm.eth --app-init-args $bounties $vault_allocations $token_honey`

Set variable:

`projects=0xc2555AbAEd3797b52248e814172d2BeA6728e542`

Now we can install the allocations app:

`dao --environment aragon:rinkeby install $onehive tps-allocations.open.aragonpm.eth --app-init-args $address_book $vault_allocations`

Set variable:

`allocations=0xdaf27F117bC69f994309430583a9c3214DCB5672`

**Set Permissions**

Address Book:

`dao --environment aragon:rinkeby acl create $onehive $address_book ADD_ENTRY_ROLE $voting_bee $voting_bee`

`dao --environment aragon:rinkeby acl create $onehive $address_book REMOVE_ENTRY_ROLE $voting_bee $voting_bee`

Vault:

`dao --environment aragon:rinkeby acl create $onehive $vault_allocations TRANSFER_ROLE $projects $voting_bee`

`dao --environment aragon:rinkeby acl grant $onehive $vault_allocations TRANSFER_ROLE $allocations`

Projects:

`dao --environment aragon:rinkeby acl create $onehive $projects CHANGE_SETTINGS_ROLE $voting_bee $voting_bee`

`dao --environment aragon:rinkeby acl create $onehive $projects ADD_REPO_ROLE $voting_bee $voting_bee`

`dao --environment aragon:rinkeby acl create $onehive $projects REMOVE_REPO_ROLE $voting_bee $voting_bee`

`dao --environment aragon:rinkeby acl create $onehive $projects CURATE_ISSUES_ROLE $dot_voting $voting_bee`

Dot voting:

`dao --environment aragon:rinkeby acl create $onehive $dot_voting CREATE_VOTES_ROLE $manager_bee $voting_bee`

`dao --environment aragon:rinkeby acl create $onehive $dot_voting ADD_CANDIDATES_ROLE $manager_bee $voting_bee`

Allocations:

`dao --environment aragon:rinkeby acl create $onehive $allocations CREATE_ACCOUNT_ROLE $voting_bee $voting_bee`

`dao --environment aragon:rinkeby acl create $onehive $allocations CREATE_ALLOCATION_ROLE $dot_voting $voting_bee`

`dao --environment aragon:rinkeby acl create $onehive $allocations EXECUTE_ALLOCATION_ROLE $manager_bee $voting_bee`
