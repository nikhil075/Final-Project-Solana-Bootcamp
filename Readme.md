# Final-Project# Solana-Bootcamp-Final-Project

## NFT ID Card Solana Smart Contract

**This is a smart contract for creating and managing Non-Fungible Token (NFT) ID Cards on the Solana blockchain. NFT ID are unique digital assets that represent identity card of a employee of a Organization which includes company name, employee id, Employee Name, Employee Phone Number, Email, their address .**

# Purpose and Functionality

The NFT Id card Smart Contract is designed to provide the following functionalities:

**Issue NFT ID Card's:** You can use this contract to issue NFT Id card to individuals. Each ID card is represented as an NFT and contains metadata such as the employee Id , employee name, and employee phone No ,their email and a link to the id card image.

**Claim ID Card :** Individuals can claim their NFT Id card once they are issued. This involves transferring the Id card NFT to their associated token account (ATA).

**Revoke NFT ID Card :** NFT ID card can be burned (destroyed) to remove them from circulation.This can be done after employee is removed from the company.

# Deployment of Solana Program 

```yaml
cidl: "0.8"
info:
  name: NFT_ID
  title: Solana NFT ID
  version: 0.0.1
  license:
    name: MIT licensed
    identifier: PrivateLicensed
types:
  NFTMetadata:
    summary: A Solana NFT ID program for issuing, transferring, and burning NFT ID card in a organization.
    solana:
      seeds:
        - name: "id"
        - name: mint
          type: sol:pubkey
    fields:
      - name: E_id
        type: u32
      - name: company_name
        type: string
        solana:
          attributes: [ cap:16 ]
      - name: department_name
        type: string
        solana:
          attributes: [ cap:16 ]
      - name: employee_name
        type: string
        solana:
          attributes: [ cap:16 ]
      - name: emp_address
        type: string
        solana:
          attributes: [ cap:255 ]
      - name: emp_email
        type: string
        solana:
          attributes: [ cap:16 ]
      - name: emp_phone
        type: string
        solana:
          attributes: [ cap:16 ]
      - name: id_image_url
        type: rs:option<string>
        solana:
          attributes: [ cap:96 ]     
      - name: mint
        type: sol:pubkey
      - name: assoc_account
        type: rs:option<sol:pubkey>
methods:
  - name: issue_id_card
    uses:
      - csl_spl_token.initialize_mint2
      - csl_spl_assoc_token.create
      - csl_spl_token.mint_to
      - csl_spl_token.set_authority
    inputs:
      - name: mint
        type: csl_spl_token.Mint
        solana:
          attributes: [ init ]
      - name: id
        type: NFTMetadata
        solana:
          attributes: [ init ]
          seeds:
            mint: mint
      - name: E_id
        type: u32
      - name: employee_name
        type: string
      - name: emp_address
        type: string
      - name: emp_email
        type: string  
  - name: claim_id_card
    uses:
      - csl_spl_assoc_token.create
      - csl_spl_token.transfer_checked
    inputs:
      - name: mint
        type: csl_spl_token.Mint
      - name: id
        type: NFTMetadata
        solana:
          attributes: [ mut ]
          seeds:
            mint: mint
  - name: revoke_id_card
    uses:
      - csl_spl_token.burn
    inputS:
      - name: mint
        type: csl_spl_token.Mint
      - name: id
        type: NFTMetadata
        solana:
          attributes: [ mut ]
          seeds:
            mint: mint

```

# To generate code using this command

```javascript
    codigo solana generate nft_certificate.yaml
```

# Step 1: Issue certificate Function

```javascript
pub fn issue_id_card(
	program_id: &Pubkey,
	for_initialize_mint_2: &[&AccountInfo],
	for_create: &[&AccountInfo],
	for_mint_to: &[&AccountInfo],
	for_set_authority: &[&AccountInfo],
	mint: &Account<spl_token::state::Mint>,
	id: &mut AccountPDA<Nftmetadata>,
	funding: &AccountInfo,
	assoc_token_account: &AccountInfo,
	wallet: &AccountInfo,
	owner: &AccountInfo,
	e_id: u32,
	employee_name: String,
	emp_address: String,
	emp_email: String,
   // id_image_url: String,
) -> ProgramResult {
    // Implement your business logic here...
   
    id.data.e_id = e_id;
    id.data.employee_name = employee_name;
    id.data.emp_address= emp_address;
    id.data.emp_email = emp_email;
 //  id.data.id_image_url = Some(id_image_url);
   id.data.mint = *mint.info.key;

 id.data.assoc_account = Some(*assoc_token_account.key);



	csl_spl_token::src::cpi::initialize_mint_2(
		for_initialize_mint_2,
		0
		, *wallet.key , None
	)?;

	csl_spl_assoc_token::src::cpi::create(
		for_create,
	)?;

	csl_spl_token::src::cpi::mint_to(
		for_mint_to,
		Default::default(),
	)?;

	csl_spl_token::src::cpi::set_authority(
		for_set_authority,
		Default::default(),
		Some(*owner.key),
	)?;


    Ok(())
}

```

# Step 2: Implement Claim certificate 

```javascript
pub fn claim_id_card(
	program_id: &Pubkey,
	for_create: &[&AccountInfo],
	for_transfer_checked: &[&AccountInfo],
	mint: &Account<spl_token::state::Mint>,
	id: &mut AccountPDA<Nftmetadata>,
	funding: &AccountInfo,
	assoc_token_account: &AccountInfo,
	wallet: &AccountInfo,
	source: &AccountInfo,
	destination: &AccountInfo,
	authority: &AccountInfo,
) -> ProgramResult {
    // Implement your business logic here...
    id.data.assoc_account = Some(*destination.key);




	csl_spl_assoc_token::src::cpi::create(
		for_create,
	)?;

	csl_spl_token::src::cpi::transfer_checked(
		for_transfer_checked,
		1,
        0,
	)?;


    Ok(())
}
    
```

# Step 3: Implement Burn certificate

```javascript
  ub fn revoke_id_card(
	program_id: &Pubkey,
	for_burn: &[&AccountInfo],
	mint: &Account<spl_token::state::Mint>,
	id: &mut AccountPDA<Nftmetadata>,
	account: &AccountPDA<spl_token::state::Account>,
	owner: &AccountInfo,
	wallet: &AccountInfo,
) -> ProgramResult {
    // Implement your business logic here...
    id.data.assoc_account = None;


	csl_spl_token::src::cpi::burn(
		for_burn,
		1,
	)?;


    Ok(())
}
```

# Build Solana program using command

```javascript
   cd program // go to program directory
   cargo build-sbf // build solana program

```

# Setup the Solana devne config file 

```javascript
solana config set --url devnet
solana airdrop 1 // to fund solana token
```

#  Deploy your program using the following command:
```javascript
solana program deploy target/deploy/nft_id.so

```

**After deploy solana program you will get program ID**
```javascript 
Program Id:  uS3s6RAhdz3GtrzK6JA5dCEfNAQKF1DgVEP7ekFCAhu
```

# Install Required Dependencies
```javascript
cd program_client //go to program_client directory
yarn add @solana/spl-token ts-node
```

# Here is the app.ts file for test functionality with frontend

```javascript


import {Connection, Keypair, PublicKey, sendAndConfirmTransaction, SystemProgram, Transaction,} from "@solana/web3.js";
import * as fs from "fs/promises";
import * as path from "path";
import * as os from "os";
import {
    revokeIdCardSendAndConfirm,
    CslSplTokenPDAs,
    deriveNftmetadataPDA,
    getNftmetadata,
    initializeClient,
    issueIdCardSendAndConfirm,
    claimIdCardSendAndConfirm,
    revokeIdCard,
} from "./index";
import {getMinimumBalanceForRentExemptAccount, getMint, TOKEN_PROGRAM_ID,} from "@solana/spl-token";

async function main(feePayer: Keypair) {
    const args = process.argv.slice(2);
    const connection = new Connection("https://api.devnet.solana.com", {
        commitment: "confirmed",
    });

    const progId = new PublicKey(args[0]!);

    initializeClient(progId, connection);


    /**
     * Create a keypair for the mint
     */
    const mint = Keypair.generate();
    console.info("Issue NFT Id card");
    console.info(mint.publicKey.toBase58());

    /**
     * Create two wallets
     */
    const employee = Keypair.generate();
    console.info("+==== Employee Wallet ====+");
    console.info(employee.publicKey.toBase58());

    const Admin = Keypair.generate();
    console.info("Adminstrator (Issuer) Wallet");
    console.info(Admin.publicKey.toBase58());

    const rent = await getMinimumBalanceForRentExemptAccount(connection);
    await sendAndConfirmTransaction(
        connection,
        new Transaction()
            .add(
                SystemProgram.createAccount({
                    fromPubkey: feePayer.publicKey,
                    newAccountPubkey: employee.publicKey,
                    space: 0,
                    lamports: rent,
                    programId: SystemProgram.programId,
                }),
            )
            .add(
                SystemProgram.createAccount({
                    fromPubkey: feePayer.publicKey,
                    newAccountPubkey: Admin.publicKey,
                    space: 0,
                    lamports: rent,
                    programId: SystemProgram.programId,
                }),
            ),
        [feePayer, employee, Admin],
    );

    /**
     * NFT ID MetaData
     */
    const [IDpub] = deriveNftmetadataPDA(
        {
            mint: mint.publicKey,
        },
        progId,
    );
    console.info("+==== Id card Metadata Address ====+");
    console.info(IDpub.toBase58());

    /**
     * Derive the John Doe's Associated Token Account, this account will be
     * holding the minted NFT.
     */
    const [EmployeeATA] = CslSplTokenPDAs.deriveAccountPDA({
        wallet: employee.publicKey,
        mint: mint.publicKey,
        tokenProgram: TOKEN_PROGRAM_ID,
    });
    console.info("+==== Employee ATA ====+");
    console.info(EmployeeATA.toBase58());

    /**
     * Derive the Jane Doe's Associated Token Account, this account will be
     * holding the minted NFT when John Doe transfer it
     */
    const [AdminATA] = CslSplTokenPDAs.deriveAccountPDA({
        wallet: Admin.publicKey,
        mint: mint.publicKey,
        tokenProgram: TOKEN_PROGRAM_ID,
    });
    console.info("+==== Issuer ATA ====+");
    console.info(AdminATA.toBase58());

    /**
     * Mint a new NFT into John's wallet (technically, the Associated Token Account)
     */
    console.info("+==== Issue ID Card. ====+");
    await issueIdCardSendAndConfirm({
        wallet: employee.publicKey,
        assocTokenAccount: EmployeeATA,
        //certificateImageUrl: "https://nftstorage.link/ipfs/bafybeifxizqlkd6yk5dbwimbykpyhaut4uk3ye6vvpnvqykinhecotnbxe/JohnDoe.png",
        eId: 4294967295,
        empEmail: "nikhil@Organization.com",
        employeeName: "Nikhil Taneja",
        empAddress: "VPO Kahnaur, ROhtak, Haryana",
        signers: {
            feePayer: feePayer,
            funding: feePayer,
            mint: mint,
            owner: employee,
        },
    });
    console.info("+==== Minted ====+");

    /**
     * Get the minted token
     */
    let issueAccount = await getMint(connection, mint.publicKey);
    console.info("+==== Issued ====+");
    console.info(issueAccount);

    /**
     * Get the ID Metadata
     */
    let IDMetadata = await getNftmetadata(IDpub);
    console.info("+==== id Card Metadata ====+");
    console.info(IDMetadata);
    console.assert(IDMetadata!.assocAccount!.toBase58(), EmployeeATA.toBase58());

    /**
     * Employee Claiming the ID Card (technically, the Associated Token Account)
     */
    console.info("+==== Transferring... ====+");
    await claimIdCardSendAndConfirm({
        wallet: Admin.publicKey,
        assocTokenAccount: AdminATA,
        mint: mint.publicKey,
        source: EmployeeATA,
        destination: AdminATA,
        signers: {
            feePayer: feePayer,
            funding: feePayer,
            authority: employee,
        },
    });
    console.info("+==== Sent to Employee ====+");

    /**
     * Get the minted token
     */
    issueAccount = await getMint(connection, mint.publicKey);
    console.info("+==== Issued ====+");
    console.info(issueAccount);

    /**
     * Get the IDMetadata Metadata
     */
    IDMetadata = await getNftmetadata(IDpub);
    console.info("+==== ID Card Metadata ====+");
    console.info(IDMetadata);
    console.assert(IDMetadata!.assocAccount!.toBase58(), AdminATA.toBase58());

    /**
     * Burn the NFT
     */
    console.info("+==== Burning... ====+");
    await revokeIdCardSendAndConfirm({
        mint: mint.publicKey,
        wallet: Admin.publicKey,
        signers: {
            feePayer: feePayer,
            owner: Admin,
        },
    });
    console.info("+==== Burned ====+");

    /**
     * Get the minted token
     */
    issueAccount = await getMint(connection, mint.publicKey);
    console.info("+==== Mint ====+");
    console.info(issueAccount);

    /**
     * Get the IDMetadata Metadata
     */
    IDMetadata = await getNftmetadata(IDpub);
    console.info("+==== IDMetadata Metadata ====+");
    console.info(IDMetadata);
    console.assert(typeof IDMetadata!.assocAccount, "undefined");
}

fs.readFile(path.join(os.homedir(), ".config/solana/id.json")).then((file) =>
    main(Keypair.fromSecretKey(new Uint8Array(JSON.parse(file.toString())))),
);

```
# Run the app.ts file with Program ID

```javascript
 npx ts-node app.ts program Id  
```

# Interacting with the Contract

You can interact with the NFT ID card Smart Contract

1. **Issue NFT ID Card**
To issue a new NFT certificate, use the **issueIdCardSendAndConfirm** function, providing details such as Employee Name,  Employee address,employee mail and id card image URL.

2. **Claim NFT ID Card**
Individuals can claim their issued NFT ID Card using the **claimIdCardSendAndConfirm** function by specifying the associated token account and providing necessary authorization.


4. **Burn NFT ID Card**
NFT ID Card can be revoked using the **revokeIdCardSendAndConfirm** function to permanently remove them from organization.

**Here are the top resources to learn Solana:** <br> 
[Rise In](https://www.risein.com/)<br> 
[Solana](https://solana.com/)<br> 
[Solana Labs](https://github.com/solana-labs)<br> 
[Codigo](https://docs.codigo.ai/)<br> 

<p align="center">
  Made with ❤️ by Nikhil Taneja
</p>