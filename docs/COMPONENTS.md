# Component Reference Documentation

## Table of Contents

1. [Syntax Module](#syntax-module)
2. [Semantics Module](#semantics-module)
3. [Substitution Module](#substitution-module)
4. [Axioms Module](#axioms-module)
5. [Checker Module](#checker-module)
6. [Examples Module](#examples-module)

## Syntax Module

### Overview
The syntax module (`syntax/`) defines the abstract syntax trees for all differential dynamic logic constructs.

### File Structure

```mermaid
graph TD
    subgraph "Syntax Module Files"
        A[symbol.v] --> B[expressions.v]
        B --> C[state.v]
        C --> D[terms_util.v]
        D --> E[terms_util2.v]
        E --> F[decidability.v]
        F --> G[list_util.v]
        G --> H[vec_util.v]
        H --> I[reals_util.v]
        I --> J[symbol_lemmas.v]
        J --> K[tactics_util.v]
    end
```

### Key Data Types

#### Terms
```coq
Inductive Term : Type :=
| KTdot       : nat -> Term
| KTfuncOf    : function_symbol -> Vector.t Term (func_arity f) -> Term  
| KTnumber    : R -> Term
| KTread      : variable -> Term
| KTneg       : Term -> Term
| KTplus      : Term -> Term -> Term
| KTminus     : Term -> Term -> Term
| KTtimes     : Term -> Term -> Term
| KTdifferential : Term -> Term
```

#### Formulas
```coq
Inductive Formula : Type :=
| KFdot         : nat -> Formula
| KFpredOf      : predicate_symbol -> Vector.t Term (pred_arity p) -> Formula
| KFquantifier  : quantifier -> variable -> Formula -> Formula
| KFnot         : Formula -> Formula
| KFand         : Formula -> Formula -> Formula
| KFor          : Formula -> Formula -> Formula
| KFimply       : Formula -> Formula -> Formula
| KFequiv       : Formula -> Formula -> Formula
| KFequal       : Term -> Term -> Formula
| KFnotequal    : Term -> Term -> Formula
| KFgreaterEqual: Term -> Term -> Formula
| KFgreater     : Term -> Term -> Formula
| KFbox         : Program -> Formula -> Formula
| KFdiamond     : Program -> Formula -> Formula
```

#### Programs
```coq
Inductive Program : Type :=
| KPconstant   : program_constant -> Program
| KPassign     : variable -> Term -> Program
| KPassignAny  : variable -> Program
| KPtest       : Formula -> Program
| KPchoice     : Program -> Program -> Program
| KPcompose    : Program -> Program -> Program
| KPloop       : Program -> Program
| KPodeSystem  : ODE -> Formula -> Program
```

#### ODEs
```coq
Inductive ODE : Type :=
| ODEconst : ode_constant -> ODE
| ODEsing  : variable -> Term -> ODE
| ODEprod  : ODE -> ODE -> ODE
```

### Syntax Tree Visualization

```mermaid
graph TB
    subgraph "Example: x := x + 1; y' = x & y >= 0"
        A[KPcompose] --> B[KPassign x]
        A --> C[KPodeSystem]
        
        B --> D[KTplus]
        D --> E[KTread x]
        D --> F[KTnumber 1]
        
        C --> G[ODEsing y]
        C --> H[KFgreaterEqual]
        
        G --> I[KTread x]
        H --> J[KTread y]
        H --> K[KTnumber 0]
    end
```

## Semantics Module

### Overview
The semantics module (`semantics/`) implements both static and dynamic semantics for differential dynamic logic.

### Architecture

```mermaid
graph TB
    subgraph "Semantics Architecture"
        A[State Space] --> B[Interpretation]
        B --> C[Term Evaluation]
        C --> D[Formula Satisfaction]
        D --> E[Program Semantics]
        E --> F[ODE Solutions]
        
        subgraph "Static Analysis"
            G[Free Variables] --> H[Bound Variables]
            H --> I[Variable Dependencies]
            I --> J[Well-formedness]
        end
        
        subgraph "Dynamic Evaluation"
            K[State Transitions] --> L[Reachability]
            L --> M[Invariants]
            M --> N[Safety Properties]
        end
    end
```

### Key Components

#### State Space
- **States**: `state = variable -> R`
- **Interpretations**: `interpretation` containing function and predicate symbols
- **Extended Interpretations**: Handle differential symbols

#### Dynamic Semantics Functions

```mermaid
sequenceDiagram
    participant S as State
    participant I as Interpretation  
    participant T as Term Evaluation
    participant F as Formula Satisfaction
    participant P as Program Execution
    
    S->>T: eval_term(t, s, i)
    T->>S: R (real value)
    
    S->>F: satisfies_formula(φ, s, i)
    F->>S: bool (satisfaction)
    
    S->>P: run_program(α, s, i)
    P->>S: set state (reachable states)
```

### Coincidence Lemmas

The coincidence lemmas establish that semantic evaluation only depends on relevant variables:

```mermaid
graph LR
    A[Formula φ] --> B[Free Variables FV(φ)]
    B --> C[States s1, s2]
    C --> D{s1 =FV(φ) s2}
    D -->|Yes| E[s1 ⊨ φ ↔ s2 ⊨ φ]
    D -->|No| F[No Guarantee]
```

## Substitution Module

### Overview
The substitution module (`substitution/`) implements uniform substitution, which is the key mechanism for axiom instantiation in differential dynamic logic.

### Uniform Substitution Architecture

```mermaid
graph TB
    subgraph "Substitution Pipeline"
        A[Input Formula] --> B[Parse Substitution σ]
        B --> C[Check Admissibility]
        C --> D{Admissible?}
        D -->|Yes| E[Apply Substitution]
        D -->|No| F[Reject]
        E --> G[Output Formula σ(φ)]
    end
    
    subgraph "Admissibility Conditions"
        H[No Variable Capture] --> I[Bound Variable Fresh]
        I --> J[Function Symbol Consistent]
        J --> K[Predicate Symbol Consistent]
    end
    
    C --> H
```

### Substitution Types

```mermaid
classDiagram
    class UniformSubstitution {
        +subst_func: function_symbol → option Term
        +subst_pred: predicate_symbol → option Formula
        +subst_ode: ode_constant → option ODE
        +subst_prog: program_constant → option Program
    }
    
    class AdmissibilityCondition {
        +no_var_capture()
        +func_symbols_consistent()
        +pred_symbols_consistent()
        +bound_vars_fresh()
    }
    
    UniformSubstitution --> AdmissibilityCondition : must satisfy
```

### Substitution Process

```mermaid
flowchart TD
    A[σ: Substitution] --> B{Term?}
    B -->|Yes| C[Substitute Functions]
    B -->|No| D{Formula?}
    
    C --> E[Check Function Admissibility]
    E --> F[Apply Function Substitution]
    
    D -->|Yes| G[Substitute Predicates & Formulas]
    D -->|No| H{Program?}
    
    G --> I[Check Predicate Admissibility]
    I --> J[Apply Predicate Substitution]
    
    H -->|Yes| K[Substitute Programs & ODEs]
    K --> L[Check Program Admissibility]
    L --> M[Apply Program Substitution]
```

## Axioms Module

### Overview
The axioms module (`axioms/`) contains the complete axiomatization of differential dynamic logic.

### Axiom Hierarchy

```mermaid
graph TB
    subgraph "Core DDL Axioms"
        A[Assignment [x:=t]φ ↔ φ_x^t] --> E[Composition [α;β]φ ↔ [α][β]φ]
        B[Test [?ψ]φ ↔ ψ→φ] --> E
        C[Choice [α∪β]φ ↔ [α]φ∧[β]φ] --> E
        D[Loop [α*]φ ↔ φ∧[α][α*]φ] --> E
        E --> F[Modal Axioms]
    end
    
    subgraph "ODE Axioms"
        G[Differential Effect DE] --> H[Differential Invariant DI]
        H --> I[Differential Cut DC]
        I --> J[Differential Ghost DG]
        J --> K[Differential Weakening DW]
    end
    
    subgraph "Meta Axioms"
        L[Uniform Substitution US] --> M[Bound Renaming BR]
        M --> N[Vacuous Quantifier VQ]
    end
    
    F --> O[Complete Calculus]
    K --> O
    N --> O
```

### ODE Axiom Details

```mermaid
sequenceDiagram
    participant DI as Differential Invariant
    participant DC as Differential Cut
    participant DG as Differential Ghost
    participant DE as Differential Effect
    
    Note over DI: [x'=f(x)&Q]P ← [x'=f(x)&Q]P'
    Note over DC: [x'=f(x)&Q]P ← [x'=f(x)&Q∧C][x'=f(x)&Q]P ∧ [x'=f(x)&Q]C
    Note over DG: [x'=f(x)&Q]P ← ∃y[x'=f(x),y'=g(x,y)&Q]P
    Note over DE: [x'=f(x)&Q](P)' ← [x'=f(x)&Q]P'
```

## Checker Module

### Overview
The checker module (`checker/`) implements a sequent calculus proof checker for differential dynamic logic.

### Proof State Architecture

```mermaid
graph TB
    subgraph "Proof State"
        A[Sequent] --> B[Hypotheses Γ]
        A --> C[Goal φ]
        B --> D[Context]
        C --> D
    end
    
    subgraph "Rule Application"
        D --> E[Select Rule]
        E --> F[Check Premises]
        F --> G[Generate Subgoals]
        G --> H[Update State]
    end
    
    subgraph "Proof Script"
        I[Step List] --> J[Step Execution]
        J --> K[State Transition]
        K --> L[Soundness Check]
    end
```

### Proof Rules

```mermaid
graph LR
    subgraph "Structural Rules"
        A[Assumption] --> B[Implication]
        B --> C[Conjunction]
        C --> D[Disjunction]
    end
    
    subgraph "Modal Rules"
        E[Box] --> F[Diamond]
        F --> G[Assignment]
        G --> H[ODE Rules]
    end
    
    subgraph "Quantifier Rules"
        I[Universal] --> J[Existential]
        J --> K[Instantiation]
    end
```

## Examples Module

### Overview
The examples module (`examples/`) provides concrete proofs demonstrating the use of the proof checker.

### Example Structure

```mermaid
graph TB
    subgraph "Example 1: Safety Property"
        A[v≥0∧A≥0] --> B[Safety Proof]
        B --> C[[x'=v,v'=A&true] v≥0]
    end
    
    subgraph "Example 2: Differential Invariant"
        D[x²+y²=1] --> E[DI Proof]
        E --> F[[x'=-y,y'=x&true] x²+y²=1]
    end
    
    subgraph "Proof Steps"
        G[step_imply_right] --> H[step_and_left]
        H --> I[step_DIgeD]
        I --> J[step_OC]
        J --> K[step_assumption]
    end
```

### Proof Script Example

```coq
apply (apply_script_preserves_soundness
  [step_imply_right "x",
   step_and_left "x" "y", 
   step_DIgeD "z" [USE_function f, USE_pred p] [],
   step_assumption "x"]).
```

This component reference provides detailed documentation of each module's structure, key components, and interactions within the OCoq-dL system.