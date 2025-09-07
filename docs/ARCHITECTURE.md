# OCoq-dL Technical Architecture Documentation

## Table of Contents

1. [System Overview](#system-overview)
2. [Architecture Components](#architecture-components)
3. [Module Dependencies](#module-dependencies)
4. [Data Flow](#data-flow)
5. [Core Components](#core-components)
6. [Proof Workflow](#proof-workflow)
7. [Extension Guidelines](#extension-guidelines)
8. [Build System](#build-system)

## System Overview

OCoq-dL is a formal verification system that implements differential dynamic logic (dL) in Coq. It provides a complete formalization of KeYmaeraX's core, including syntax, semantics, uniform substitution, axioms, and a proof checker for differential dynamic logic.

```mermaid
graph TB
    subgraph "OCoq-dL System Architecture"
        A[User Proofs] --> B[Proof Checker]
        B --> C[Axioms & Rules]
        B --> D[Uniform Substitution]
        D --> E[Semantics]
        E --> F[Syntax]
        
        subgraph "Core Logic"
            C --> G[DDL Axioms]
            C --> H[ODE Axioms]
            C --> I[Differential Invariants]
        end
        
        subgraph "Semantic Foundation"
            E --> J[Dynamic Semantics]
            E --> K[Static Semantics]
            E --> L[Coincidence Lemmas]
        end
        
        subgraph "Syntactic Foundation"
            F --> M[Terms]
            F --> N[Formulas]
            F --> O[Programs]
            F --> P[ODEs]
        end
    end
```

## Architecture Components

The system is organized into seven main modules, each with specific responsibilities:

### 1. Syntax Module
**Location**: `syntax/`  
**Purpose**: Defines the abstract syntax trees for differential dynamic logic

### 2. Semantics Module  
**Location**: `semantics/`  
**Purpose**: Implements static and dynamic semantics for dL constructs

### 3. Substitution Module
**Location**: `substitution/`  
**Purpose**: Implements uniform substitution and proves its correctness

### 4. Axioms Module
**Location**: `axioms/`  
**Purpose**: Contains DDL axioms, ODE axioms, and differential invariant rules

### 5. Checker Module
**Location**: `checker/`  
**Purpose**: Implements sequent calculus and proof verification

### 6. Examples Module
**Location**: `examples/`  
**Purpose**: Provides example proofs and verification cases

### 7. Utilities Module
**Location**: `coq-tools/`  
**Purpose**: Contains utility tactics and helper functions

## Module Dependencies

```mermaid
graph TD
    subgraph "Dependency Graph"
        Syntax[syntax] --> Semantics[semantics]
        Semantics --> Substitution[substitution]
        Substitution --> Axioms[axioms]
        Axioms --> Checker[checker]
        Checker --> Examples[examples]
        
        subgraph "Syntax Files"
            S1[symbol.v] --> S2[expressions.v]
            S2 --> S3[state.v]
            S3 --> S4[terms_util.v]
        end
        
        subgraph "Semantics Files"
            SE1[semantics_util.v] --> SE2[static_sem.v]
            SE2 --> SE3[dynamic_semantics.v]
            SE3 --> SE4[coincidence.v]
        end
        
        subgraph "Substitution Files"
            SU1[US_defs.v] --> SU2[US.v]
            SU2 --> SU3[soundness.v]
        end
        
        subgraph "Axioms Files"
            A1[differential.v] --> A2[DDLaxioms.v]
            A2 --> A3[differential_axioms.v]
        end
    end
```

## Data Flow

The following diagram shows how data flows through the system from syntax to verified proofs:

```mermaid
flowchart TD
    subgraph "Input Layer"
        A[dL Formula] --> B[Parse Syntax]
        A1[dL Program] --> B
        A2[dL Term] --> B
    end
    
    subgraph "Syntax Layer"
        B --> C[Abstract Syntax Tree]
        C --> D[Term Structures]
        C --> E[Formula Structures]
        C --> F[Program Structures]
    end
    
    subgraph "Semantic Layer"
        D --> G[Term Semantics]
        E --> H[Formula Semantics]
        F --> I[Program Semantics]
        
        G --> J[Static Analysis]
        H --> J
        I --> J
        
        J --> K[Semantic Validity]
    end
    
    subgraph "Substitution Layer"
        K --> L[Uniform Substitution]
        L --> M[Substitution Validity]
        M --> N[Bound Variable Handling]
    end
    
    subgraph "Proof Layer"
        N --> O[Axiom Application]
        O --> P[Rule Application]
        P --> Q[Proof Checking]
        Q --> R[Verified Theorem]
    end
```

## Core Components

### Syntax Component (`syntax/`)

The syntax module defines the fundamental structures of differential dynamic logic.

```mermaid
classDiagram
    class Term {
        +KTdot: nat → Term
        +KTfuncOf: function_symbol → Vector Term → Term
        +KTnumber: R → Term
        +KTread: variable → Term
        +KTneg: Term → Term
        +KTplus: Term → Term → Term
        +KTminus: Term → Term → Term
        +KTtimes: Term → Term → Term
        +KTdifferential: Term → Term
    }
    
    class Formula {
        +KFdot: nat → Formula
        +KFpredOf: predicate_symbol → Vector Term → Formula
        +KFquantifier: quantifier → variable → Formula → Formula
        +KFnot: Formula → Formula
        +KFand: Formula → Formula → Formula
        +KFor: Formula → Formula → Formula
        +KFimply: Formula → Formula → Formula
        +KFequiv: Formula → Formula → Formula
        +KFequal: Term → Term → Formula
        +KFnotequal: Term → Term → Formula
        +KFgreaterEqual: Term → Term → Formula
        +KFgreater: Term → Term → Formula
        +KFbox: Program → Formula → Formula
        +KFdiamond: Program → Formula → Formula
    }
    
    class Program {
        +KPconstant: program_constant → Program
        +KPassign: variable → Term → Program
        +KPassignAny: variable → Program
        +KPtest: Formula → Program
        +KPchoice: Program → Program → Program
        +KPcompose: Program → Program → Program
        +KPloop: Program → Program
        +KPodeSystem: ODE → Formula → Program
    }
    
    class ODE {
        +ODEconst: ode_constant → ODE
        +ODEsing: variable → Term → ODE
        +ODEprod: ODE → ODE → ODE
    }
    
    Term --|> Formula : used in
    Program --|> Formula : box/diamond
    ODE --|> Program : ODE systems
```

### Semantics Component (`semantics/`)

```mermaid
graph TB
    subgraph "Static Semantics"
        A[Free Variables] --> B[Bound Variables]
        B --> C[Variable Dependencies]
        C --> D[Well-formedness]
    end
    
    subgraph "Dynamic Semantics"
        E[State] --> F[Interpretation]
        F --> G[Term Evaluation]
        G --> H[Formula Satisfaction]
        H --> I[Program Execution]
        I --> J[ODE Solutions]
    end
    
    subgraph "Correctness Properties"
        D --> K[Coincidence Lemmas]
        J --> K
        K --> L[Semantic Soundness]
    end
```

### Substitution Component (`substitution/`)

The uniform substitution system is central to the calculus:

```mermaid
sequenceDiagram
    participant User as User Input
    participant Parser as Substitution Parser
    participant US as Uniform Substitution
    participant Checker as Admissibility Checker
    participant Result as Substituted Formula
    
    User->>Parser: Input substitution σ
    Parser->>US: Parse substitution rules
    US->>Checker: Check admissibility conditions
    Checker->>Checker: Verify no variable capture
    Checker->>US: Admissibility result
    US->>Result: Apply substitution σ(φ)
    Result->>User: Substituted formula
```

### Axioms Component (`axioms/`)

```mermaid
graph TB
    subgraph "DDL Axioms"
        A[Assignment Axiom] --> D[Composition]
        B[Test Axiom] --> D
        C[Choice Axiom] --> D
        D --> E[Loop Axiom]
        E --> F[Box/Diamond Duality]
    end
    
    subgraph "ODE Axioms"
        G[Differential Effect] --> H[Differential Invariant]
        H --> I[Differential Cut]
        I --> J[Differential Ghost]
        J --> K[ODE Invariant]
    end
    
    subgraph "Meta Axioms"
        L[Uniform Substitution] --> M[Bound Renaming]
        M --> N[Vacuous Quantifier]
    end
    
    F --> O[Complete Calculus]
    K --> O
    N --> O
```

## Proof Workflow

The proof construction and verification follows a structured workflow:

```mermaid
stateDiagram-v2
    [*] --> ParseGoal
    ParseGoal --> ValidateGoal : syntax check
    ValidateGoal --> ChooseRule : goal is well-formed
    ValidateGoal --> Error : syntax error
    
    ChooseRule --> ApplyAxiom : axiom applicable
    ChooseRule --> ApplyRule : proof rule applicable
    ChooseRule --> ApplySubstitution : substitution needed
    
    ApplyAxiom --> CheckAxiomConditions
    ApplyRule --> CheckRuleConditions  
    ApplySubstitution --> CheckAdmissibility
    
    CheckAxiomConditions --> Success : conditions met
    CheckAxiomConditions --> Backtrack : conditions failed
    
    CheckRuleConditions --> GenerateSubgoals : conditions met
    CheckRuleConditions --> Backtrack : conditions failed
    
    CheckAdmissibility --> SubstituteAndContinue : admissible
    CheckAdmissibility --> Backtrack : not admissible
    
    GenerateSubgoals --> ParseGoal : for each subgoal
    SubstituteAndContinue --> ParseGoal : continue with substituted goal
    
    Success --> [*]
    Backtrack --> ChooseRule : try different approach
    Error --> [*]
```

### Proof Checker Architecture

```mermaid
graph TB
    subgraph "Proof State Management"
        A[Sequent] --> B[Hypotheses]
        A --> C[Goal Formula]
        B --> D[Context Management]
        C --> D
    end
    
    subgraph "Rule Application"
        D --> E[Rule Selector]
        E --> F[Premise Checking]
        F --> G[Conclusion Generation]
        G --> H[State Update]
    end
    
    subgraph "Verification"
        H --> I[Soundness Check]
        I --> J[Completeness Check]
        J --> K[Proof Certificate]
    end
```

## Extension Guidelines

### Adding New Axioms

1. Define the axiom in `axioms/DDLaxioms.v` or `axioms/differential_axioms.v`
2. Prove soundness with respect to the semantics
3. Add the axiom to the proof checker in `checker/checker.v`
4. Create examples demonstrating usage

### Adding New Language Constructs

1. Extend syntax in `syntax/expressions.v`
2. Define semantics in `semantics/dynamic_semantics.v` and `semantics/static_sem.v`
3. Extend uniform substitution in `substitution/US.v`
4. Prove coincidence lemmas in `semantics/coincidence.v`
5. Add corresponding axioms if needed

### Extending the Proof Checker

1. Define new proof rules in appropriate axiom files
2. Implement rule application logic in `checker/checker.v`
3. Prove soundness of new rules
4. Add examples and tests

## Build System

The project uses a custom build system based on Coq's dependency analysis:

```mermaid
graph LR
    subgraph "Build Process"
        A[create_makefile.sh] --> B[Dependency Analysis]
        B --> C[Makefile Generation]
        C --> D[Parallel Compilation]
        D --> E[all.vo Target]
    end
    
    subgraph "Dependencies"
        F[Coq 8.9.1] --> G[Coquelicot 3.0.2]
        G --> H[SSReflect]
        H --> I[Standard Library]
    end
    
    E --> J[Verification Complete]
```

### Build Commands

```bash
# Generate Makefile and build
./create_makefile.sh
make -j <cores>

# Clean build artifacts
make clean

# Build specific component
make syntax/expressions.vo
```

The architecture supports modular development and verification, with clear separation of concerns between syntax, semantics, substitution, axioms, and proof checking. Each component builds upon the previous ones, creating a layered architecture that ensures correctness at each level.