# story-goldysky-docs
story-goldysky-docs
===================

Deploying Subgraphs to Goldsky This guide provides step-by-step instructions on how to create a subgraph, including defining the schema and mappings, and then deploying it to Goldsky. This is intended to help developers on our team set up and deploy subgraphs for contracts deployed on the story-testnet network.

Table of Contents Prerequisites Step 1: Install Goldsky CLI and Log In Step 2: Set Up Your Subgraph Create subgraph.yaml Define the Schema Implement Mappings Step 3: Generate Code Step 4: Deploy to Goldsky Additional Resources Conclusion Prerequisites Before you begin, ensure you have the following installed on your machine:

Node.js and Yarn Git The Graph CLI Install The Graph CLI globally:

bash Copy code npm install -g @graphprotocol/graph-cli Step 1: Install Goldsky CLI and Log In Install Goldsky CLI Run the following command to install the Goldsky CLI:

bash Copy code curl [https://goldsky.com](https://goldsky.com/) | sh Log In to Goldsky Create an API Key:

Go to your Goldsky project's Settings page. Create an API key. Log In via CLI:

In your terminal, run:

bash Copy code goldsky login Paste your API key when prompted.

Verify Installation:

Run the Goldsky CLI to ensure it's working:

bash Copy code goldsky Step 2: Set Up Your Subgraph Clone the Repository Clone your fork of the subgraph repository:

bash Copy code git clone [git@github.com](mailto:git@github.com):/v3-subgraph.git cd v3-subgraph Install Dependencies bash Copy code yarn install Create subgraph.yaml Create a subgraph.yaml file in the root directory of your project. Here's a base example tailored for the contract at 0x1E6678e0AF0Aa0220a01fCA979Fa3612e3E4e2ce on story-testnet:

yaml Copy code specVersion: 0.0.4 description: Subgraph for the Uniswap V3-like contract on story-testnet. repository: [https://github.com/](https://github.com/)/v3-subgraph schema: file: ./schema.graphql features:

*   nonFatalErrors
    
*   grafting
    

dataSources:

*   kind: ethereum/contract name: Factory network: story-testnet source: abi: Factory address: '0x1E6678e0AF0Aa0220a01fCA979Fa3612e3E4e2ce' startBlock: mapping: kind: ethereum/events apiVersion: 0.0.7 language: wasm/assemblyscript file: ./src/mappings/factory.ts entities: - Pool - Token abis: - name: Factory file: ./abis/Factory.json - name: ERC20 file: ./abis/ERC20.json eventHandlers: - event: PoolCreated(indexed address,indexed address,indexed uint24,int24,address) handler: handlePoolCreated
    

templates:

*   kind: ethereum/contract name: Pool network: story-testnet source: abi: Pool mapping: kind: ethereum/events apiVersion: 0.0.7 language: wasm/assemblyscript file: ./src/mappings/pool/index.ts entities: - Pool - Token abis: - name: Pool file: ./abis/Pool.json - name: Factory file: ./abis/Factory.json - name: ERC20 file: ./abis/ERC20.json eventHandlers: - event: Initialize(uint160,int24) handler: handleInitialize - event: Swap(indexed address,indexed address,int256,int256,uint160,uint128,int24) handler: handleSwap - event: Mint(address,indexed address,indexed int24,indexed int24,uint128,uint256,uint256) handler: handleMint - event: Burn(indexed address,indexed int24,indexed int24,uint128,uint256,uint256) handler: handleBurn - event: Collect(indexed address,address,indexed int24,indexed int24,uint128,uint128) handler: handleCollect Customize subgraph.yaml Replace with the block number where your contract was deployed. Update repository with your repository URL. Ensure all ABIs referenced exist in the ./abis directory. Adjust entities and eventHandlers as per your contract's specifications. Define the Schema Create a schema.graphql file in the root directory. Define your entities based on the data you want to index. For example:
    

graphql Copy code type Pool @entity { id: ID! token0: Token! token1: Token! feeTier: BigInt!

Add other fields as necessary
=============================

}

type Token @entity { id: ID! symbol: String! name: String! decimals: BigInt!

Add other fields as necessary
=============================

} Implement Mappings Mappings are TypeScript functions that transform Ethereum events into entity data. Implement the handlers specified in your subgraph.yaml.

Example: src/mappings/factory.ts typescript Copy code import { PoolCreated } from '../../generated/Factory/Factory'; import { Pool, Token } from '../../generated/schema';

export function handlePoolCreated(event: PoolCreated): void { // Load or create Token entities let token0 = Token.load(event.params.token0.toHexString()); if (token0 == null) { token0 = new Token(event.params.token0.toHexString()); // Set token0 fields token0.save(); }

let token1 = Token.load(event.params.token1.toHexString()); if (token1 == null) { token1 = new Token(event.params.token1.toHexString()); // Set token1 fields token1.save(); }

// Create a new Pool entity let pool = new Pool(event.params.pool.toHexString()); pool.token0 = token0.id; pool.token1 = token1.id; pool.feeTier = event.params.fee; // Set other Pool fields pool.save();

// Create a new Pool template instance // This allows indexing of events from the new Pool contract PoolTemplate.create(event.params.pool); } Example: src/mappings/pool/index.ts Implement the handlers for the Pool contract events, such as handleSwap, handleMint, etc.

Step 3: Generate Code Run the following command to generate TypeScript types from your schema and ABIs:

bash Copy code graph codegen This command generates code in the generated/ directory.

Step 4: Deploy to Goldsky Deploy the Subgraph Use the Goldsky CLI to deploy your subgraph:

bash Copy code goldsky subgraph deploy / Replace with a name for your subgraph. Replace with a version identifier, like v1.0.0. Monitor Deployment Log in to your Goldsky dashboard. Navigate to your project and monitor the indexing status. Use the GraphQL playground provided by Goldsky to test queries. Additional Resources Goldsky Documentation: [https://docs.goldsky.com/](https://docs.goldsky.com/) The Graph Documentation: [https://thegraph.com/docs/](https://thegraph.com/docs/) AssemblyScript Documentation: [https://www.assemblyscript.org/](https://www.assemblyscript.org/) Conclusion By following this guide, you should be able to set up a subgraph, define the necessary schema and mappings, and deploy it to Goldsky. This will enable efficient querying of blockchain data for your applications.
