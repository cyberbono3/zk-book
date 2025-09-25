```graph TB
  A["Approaches to Practical SNARK Design"]
  A --> B["Polynomial IOP + Polynomial Commitment Scheme"]
  A --> C["Linear PCP + Pairing-based cryptography"]
  C --> C1["Groth16"]

  B --> B1["Polynomial IOP approaches"]
  B --> B2["Polynomial Commitment Approaches"]

  B1 --> D1["IP-based"]
  B1 --> D2["MIP-based"]
  B1 --> D3["Constant-round polynomial IOP"]

  D1 --> GKR["GKR protocol"]
  D2 --> Clover["Clover"]
  D2 --> Spartan["Spartan"]
  D3 --> Marlin["Marlin"]
  D3 --> PLONK["PLONK"]

  B2 --> E1["IOP-based"]
  B2 --> E2["Discrete-log-based (transparent)"]
  B2 --> E3["Pairing-based (non-transparent)"]

  E1 --> FRI["FRI"]
  E1 --> Ligero["Ligero/Brakedown-commit"]

  E2 --> Hyax["Hyax-commit"]
  E2 --> Bulletproofs["Bulletproofs"]
  E2 --> Dory["Dory"]

  E3 --> KZG["KZG"]
```
