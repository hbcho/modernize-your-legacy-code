# COBOL Account Management

This directory documents the COBOL account-management example in `src/cobol/`.
The program provides a menu for viewing a balance, crediting an account, and
debiting an account.

## Source Files

- `src/cobol/main.cob` (`MainProgram`) displays the menu, reads the user's
  selection, and dispatches balance, credit, and debit requests to `Operations`.
  Selecting option 4 exits the program; other unrecognized selections display
  an error message.
- `src/cobol/operations.cob` (`Operations`) implements the account actions.
  It reads the current balance for display, prompts for an amount when crediting
  or debiting, and delegates balance reads and updates to `DataProgram`.
- `src/cobol/data.cob` (`DataProgram`) owns the stored balance. Its `READ`
  operation copies the stored balance to the caller, and `WRITE` replaces the
  stored balance with the caller-provided value.

## Student Account Rules

- The example currently models one shared account balance, initialized to
  `1000.00`. It does not store student names, IDs, or separate balances per
  student.
- A credit adds the entered amount to the stored balance.
- A debit is accepted only when the current balance is greater than or equal to
  the entered amount. Otherwise, the balance is unchanged and an insufficient
  funds message is displayed.
- The code does not validate that entered amounts are positive. It also does
  not persist the balance outside the running program.

## Sequence Diagram

```mermaid
sequenceDiagram
  actor User
  participant Main as MainProgram
  participant Ops as Operations
  participant Data as DataProgram

  loop Until the user exits
    Main->>User: Display account menu
    User->>Main: Enter menu choice
    alt View balance (1)
      Main->>Ops: CALL TOTAL
      Ops->>Data: CALL READ, FINAL-BALANCE
      Data-->>Ops: Return stored balance
      Ops->>User: Display current balance
    else Credit account (2)
      Main->>Ops: CALL CREDIT
      Ops->>User: Prompt for credit amount
      User->>Ops: Enter amount
      Ops->>Data: CALL READ, FINAL-BALANCE
      Data-->>Ops: Return stored balance
      Ops->>Ops: Add amount to balance
      Ops->>Data: CALL WRITE, FINAL-BALANCE
      Data-->>Ops: Store updated balance
      Ops->>User: Display new balance
    else Debit account (3)
      Main->>Ops: CALL DEBIT
      Ops->>User: Prompt for debit amount
      User->>Ops: Enter amount
      Ops->>Data: CALL READ, FINAL-BALANCE
      Data-->>Ops: Return stored balance
      alt Balance is sufficient
        Ops->>Ops: Subtract amount from balance
        Ops->>Data: CALL WRITE, FINAL-BALANCE
        Data-->>Ops: Store updated balance
        Ops->>User: Display new balance
      else Insufficient funds
        Ops->>User: Display insufficient funds message
      end
    else Exit (4)
      Main->>Main: Set continue flag to NO
    else Invalid choice
      Main->>User: Display invalid choice message
    end
  end
  Main->>User: Display exit message
```