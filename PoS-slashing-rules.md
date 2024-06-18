# Ethereum
The Ethereum consensus specification is maintained as part of the Ethereum research repositories, primarily in the [eth2.0-consensus-specs](https://github.com/ethereum/consensus-specs) repository on GitHub.

## Validators

1. **Slashing**: 
   - **Purpose**: To penalize validators who exhibit malicious behavior or severe negligence.
   - **Conditions**: Validators can be slashed for double-signing (proposing or attesting to two conflicting blocks) or for proposing invalid blocks.
   - **Consequences**: When slashed, a validator loses a portion of their staked ETH and is forcibly removed from the validator set. The penalty includes a base penalty and a variable penalty proportional to the number of other validators slashed around the same time.
   - **Base Penalty**: The base penalty for slashing is 1/32 of the staked coins.
   - **Variable Penalty**: In addition to the base penalty, a variable penalty is applied. This penalty scales with the number of validators slashed within the same epoch. The formula for the total slashing penalty is:
     \[
     \text{Total Penalty} = \text{Base Penalty} + \left( \frac{\text{Stake}}{3} \times \frac{\text{Total Slashed in Epoch}}{\text{Total Validators}} \right)
     \]
   This means that if many validators are slashed simultaneously, the penalty for each increases significantly.
    - Slashing penalties are described in the [Beacon Chain](https://github.com/ethereum/consensus-specs/blob/dev/specs/phase0/beacon-chain.md) specification.
    - **Code**: [Attester Slashing](https://github.com/ethereum/consensus-specs/blob/dev/specs/phase0/beacon-chain.md#attester-slashing)
    ```
    def process_attester_slashing(state: BeaconState, attester_slashing: AttesterSlashing) -> None:
        # Additional processing logic...
        slash_validator(state, slashed_index, whistleblower_index)
    ```
 
2. **Inactivity Leak**:
   - **Purpose**: To penalize validators who fail to participate in the network due to being offline or not producing attestations.
   - **Conditions**: This mechanism is activated during periods when the network fails to finalize blocks due to a significant number of validators being offline.
   - **Consequences**: Validators gradually lose a portion of their staked ETH if they remain inactive during these periods. This loss continues until the network returns to a healthy state.
   - The penalty for inactivity is proportional to the length of the inactivity period. During an inactivity leak, validators lose a small percentage of their stake every epoch until finality is restored. The exact amount varies but is generally designed to be around 0.1% of the validator's stake per epoch of inactivity.
   - Location: `process_inactivity_updates` function in the beacon chain state transition.
   - Code: [eth2.0-specs/blob/dev/specs/phase0/beacon-chain.md#inactivity-penalty-updates](https://github.com/ethereum/consensus-specs/blob/dev/specs/phase0/beacon-chain.md#inactivity-penalty-updates)
    ```
    def process_inactivity_updates(state: BeaconState) -> None:
        for index in get_eligible_validator_indices(state):
            # Apply penalties...
            penalties[index] += base_inactivity_penalty
    ```

   
3. **Reward Reduction**:
   - **Purpose**: To incentivize continuous and correct participation.
   - **Conditions**: Validators earn rewards for proposing and attesting to blocks correctly. If they fail to perform these duties, they miss out on these rewards.
   - **Consequences**: Missing out on rewards effectively reduces a validator's overall earnings, serving as a financial penalty for non-participation.

4. **Ejection**:
   - **Purpose**: To maintain a set of active and reliable validators.
   - **Conditions**: Validators that fall below a minimum balance threshold (16 ETH from the original 32 ETH stake) due to penalties or inactivity are ejected from the validator set.
   - **Consequences**: Ejected validators stop earning rewards and must exit the network, although they can choose to rejoin by meeting the requirements again.

## Attesters
Attester slashing is a mechanism to penalize validators who make contradictory attestations. This is meant to prevent validators from trying to deceive the network by attesting to conflicting information.

1. Slashing
    - **Conditions for Slashing**: 
        - A validator signs two different attestations for the same epoch.
        - The attestations conflict in a way that violates the consensus rules.

    - **Penalty for Attester Slashing**: 
        - When a validator is slashed for a conflicting attestation, they lose a portion of their staked ETH. The penalty includes a base penalty and a variable penalty that scales with the number of other validators slashed within the same epoch.
    - **Code location**:
    **Attester Slashing (process_attester_slashing function)**: [Attester Slashing](https://github.com/ethereum/consensus-specs/blob/dev/specs/phase0/beacon-chain.md#attester-slashing)
    ```
    def process_attester_slashing(state: BeaconState, attester_slashing: AttesterSlashing) -> None:
        # Check conditions for slashing
        assert is_valid_attester_slashing(state, attester_slashing)
    
        slash_validator(state, attester_slashing.attestation_1.validator_index)
        slash_validator(state, attester_slashing.attestation_2.validator_index)
    ```
        
2. Inactivity
    - **Inactivity Penalty**:
        - Attesters also face penalties for being inactive, especially during critical periods where the network is struggling to achieve finality.

    - **Conditions for Inactivity Penalty**: 
        - When the network fails to finalize for several epochs, an inactivity leak is triggered. Validators who fail to attest during these periods are penalized.

    - **Penalty for Inactivity**: 
        - Validators gradually lose a portion of their staked ETH each epoch they remain inactive during an inactivity leak. This penalty continues until the network regains finality.
    - **Code Location**: [Inactivity Penalty Updates](https://github.com/ethereum/consensus-specs/blob/dev/specs/phase0/beacon-chain.md#inactivity-penalty-updates)
    ```
    def process_inactivity_updates(state: BeaconState) -> None:
        for index in get_eligible_validator_indices(state):
            # Apply inactivity penalties
            penalties[index] += base_inactivity_penalty
    ```





