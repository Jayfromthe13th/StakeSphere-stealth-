# StakeSphere: Security Audit Report




## Audit Summary

This report outlines the findings from a comprehensive security audit conducted for **StakeSphere**. The audit targeted the project's smart contract suite with the objective of identifying and mitigating potential vulnerabilities, thereby strengthening the security and robustness of the project's blockchain infrastructure.

## Findings Summary

The audit revealed findings categorized severity levels. Recommendations for remediation are provided to address these vulnerabilities effectively.

### Critical Findings

#### Token Identifier Collision

- **Description:**  

The get_pool_address function generates a unique address for a liquidity pool linked to trading pairs of fungible assets. This function creates and returns an address that uniquely identifies the liquidity pool for the specified pair of tokens. Users have the liberty to construct an Object<Metadata> using any symbol of their choice, which offers a great deal of flexibility. This flexibility, however, can lead to the creation of Object<Metadata> instances that closely resemble other existing instances. This situation might lead to a seed collision, which could subsequently cause a collision in the generation of the pool address.
- **Affected Code:**
  ```rust
  public fun get_pool_address(token_1: Object<Metadata>, token_2: Object<Metadata>): address {
   let token_symbol = string::utf8(b"LP-");
   string::append(&mut token_symbol, fungible_asset::symbol(token_1));
   string::append_utf8(&mut token_symbol, b"-");
   string::append(&mut token_symbol, fungible_asset::symbol(token_2));
   let seed = *string::bytes(&token_symbol);
   object::create_object_address(&@swap, seed)
}


- **Recommendation:**  
Use object::object_address returns an unique identifier for each Object<Metadata>
Ex:

  ```rust
  public fun get_pool_address(token_1: Object<Metadata>, token_2: Object<Metadata>): address {
   let seeds = vector[];
   vector::append(&mut seeds, bcs::to_bytes(&object::object_address(&token_1)));
   vector::append(&mut seeds, bcs::to_bytes(&object::object_address(&token_2)));
   object::create_object_address(&@swap, seed)
}


### High Severity Findings

#### Oracle Confidence Checks 

- **Description:**  
High oracle confidence values suggest that there is disagreement among providers about the actual price. For instance, Pyth measures confidence as the difference between the 25th and 75th quartiles and the median price.


- **Recommendation:**
Check the confidence of oracles.


### Medium Severity findings 

#### Liquidate Minimum Debt Vaults

- **Description:**  

StakeSphere enforces a minimum debt threshold when repaying vaults. That being said, liquidate_repay also enforces that the collateral ratio of the vault isn’t repaid fully. This means that vaults that are close to the minimum debt threshold cannot be liquidated.

- **Affected Code:**
  ```rust
 					
       let collateral_ratio = collateral_ratio_internal(engine, vault);
        assert!(

					
           collateral_ratio < engine.liquidation_collateral_ratio,

					
           error::invalid_argument(ELIQUIDATE_TOO_MUCH),
        );




- **Recommendation:**  
Rework the minimum collateral ratio check


Conclusion
The audit of StakeSphere uncovered  issues requiring immediate attention to protect the project's integrity and user assets. Implementing the recommended changes will significantly enhance the security framework of the project.
