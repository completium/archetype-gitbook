---
description: Health insurance example
---

# Health care

This contract is adapted from the DAML 's  [introductory insurance contract](https://docs.daml.com/getting-started/introduction.html).

DAML is a distributed private ledger and does not manage payments; the contract allows the doctor to generate bills for insurer and patient. 

The strength of a public blockchain with a cryptocurrency, is that it allows payments and accounting of payments in a _single business process_. The main differences between the contract below and the DAML version are:

* payments are executed through contract entries
* the contract enters the _Running_ state with validation from both insurer and patient parties

The main benefit of this version is that it may serve as an opposable document in the case of a default in payment: indeed, since the contract is used to channel payments, the contract is accounting for any default of payment \(in the _debts_ variables\).

For example, a doctor may go to court and exhibit the _debt_ value of his corresponding _doctor_ asset as a proof of the default in payment.

From a design point of vue, the contract manages several doctors. A doctor is registered with approvals from both patient and insurer parties.

{% hint style="info" %}
We see in this basic example, that a smart contract on a public blockchain with a cryptocurrency, enables organisations to capture the critical aspects of a business process : payments, permissions and accounting.
{% endhint %}

```ocaml
archetype health_care

constant insurer : role = @tz1Lc2qBKEWCBeDU8npG6zCeCqpmaegRi6Jg
constant patient : role = @tz1bfVgcJC4ukaQSHUe1EbrUd5SekXeP9CWk

constant fee : tez = 100tz
constant deductible : tez = 500tz
constant period : duration = 30d

variable last_fee : date = 2019-11-12

variable consultation_debt : tez = 0tz

asset doctor identified by id {
  id   : role;
  debt : tez = 0tz;
}

states =
| Created   initial
| Running
| Canceled

transition confirm () {
  from Created
  (*signed by [insrurer; patient]*)
  to Running
  with effect {
    last_fee := now
  }
}

transition cancel () {
  called by insurer or patient
  from Created
  to Canceled
}

entry register_doctor (docid : address) {
  (*signed by [insurer; patient]*)
  require {
     r1 : state = Running;
  }
  effect {
    doctor.add ({ id = docid })
  }
}

entry declare_consultation (v : tez) {
  require {
     r2 : state = Running;
     r3 : doctor.contains(caller);
  }
  effect {
    doctor.update(caller, { debt += v });
    consultation_debt += min (v, deductible)
  }
}

entry pay_doctor (docid : address) {
  specification {
    postcondition idem_balance_pay_doctor {
      balance = before.balance
    }
  }
  accept transfer
  called by insurer
  require {
    r4 : state = Running;
  }
  effect {
    var ldebt = doctor[docid].debt;
    var decrease : tez = min (transferred, ldebt);
    transfer decrease to docid;
    transfer (transferred - decrease) to insurer;
    doctor.update (docid, { debt -= decrease })
  }
}

entry pay_fee () {
  specification {
    postcondition idem_balance_pay_fee {
      balance = before.balance
    }
  }
  accept transfer
  called by patient
  require {
    r5 : state = Running;
  }
  effect {
    var nb_periods : int = (now - last_fee) div period;  (* div is euclidean *)
    var due = nb_periods * fee;
    var decrease : tez = min (transferred, due);
    transfer decrease to insurer;
    transfer (transferred - decrease) to patient;
    last_fee += nb_periods * period     (* last_fee <> now because div is euclidean *)
  }
}

entry pay_consulation () {
  specification {
    idem_balance_pay_consultation : balance = before.balance;
  }
  accept transfer
  called by patient
  require {
    r6 : state = Running;
  }
  effect {
    var decrease : tez = min (transferred, consultation_debt);
    transfer decrease to insurer;
    transfer (transferred - decrease) to patient;
    consultation_debt -= decrease
  }
}

```

