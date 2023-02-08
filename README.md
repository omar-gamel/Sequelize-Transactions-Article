# Sequelize Transactions

Atomic transactions are the backbone of relational databases. ACID-compliant databases like MySQL or Postgres provide some very important properties of data integrity with the help of transactions. This essentially means all statements executed within the scope of a single transaction are either all successfully executed or the entire block is rolled back.

<b>What does “ACID compliant” mean?</b> 
The short answer is that ACID, an acronym for “Atomicity, Consistency, Isolation, and Durability,” is a set of principles that ensure database transactions are processed reliably. When any data storage system upholds those principles, it is said to be ACID compliant.
#
Sequelize supports two ways of using transactions:
1. Unmanaged transactions
2. Managed transactions

<b>Unmanaged transactions:</b>
<br>
Committing and rolling back the transaction should be done manually by the user by calling the Sequelize transaction.commit() & transaction.rollback() methods.
<p>

```javascript
// First, we start a transaction from your connection and save it into a variable
const t = await sequelize.transaction();

try {

  // Then, we do some calls passing this transaction as an option:

  const user = await User.create({
    firstName: 'Bart',
    lastName: 'Simpson'
  }, { transaction: t });

  await user.addSibling({
    firstName: 'Lisa',
    lastName: 'Simpson'
  }, { transaction: t });

  // If the execution reaches this line, no errors were thrown.
  // We commit the transaction.
  await t.commit();

} catch (error) {

  // If the execution reaches this line, an error was thrown.
  // We rollback the transaction.
  await t.rollback();
}
```
</p>

In the above, unmanaged transaction example a User entity is created first & then an Account entry should be created. User manually needs to commit the transaction by calling commit() & if any error occurs roll back the transaction using rollback().

<b>Managed transactions:</b>
<br>
Sequelize will automatically roll back the transaction if any error is thrown, or commit the transaction otherwise.
<p>

```javascript
try {

  const result = await sequelize.transaction(async (t) => {

    const user = await User.create({
      firstName: 'Abraham',
      lastName: 'Lincoln'
    }, { transaction: t });

    await user.setShooter({
      firstName: 'John',
      lastName: 'Boothe'
    }, { transaction: t });

    return user;

  });

  // If the execution reaches this line, the transaction has been committed successfully
  // `result` is whatever was returned from the transaction callback (the `user`, in this case)

} catch (error) {

  // If the execution reaches this line, an error occurred.
  // The transaction has already been rolled back automatically by Sequelize!

}
```
</p>

In the above, the transaction is started by passing a callback to `sequelize.transaction` which generated transaction object "t". Sequelize executes the callback, passing `t` into it. If the callback throws an error, Sequelize will automatically roll back the transaction & If the callback succeeds, Sequelize will automatically commit the transaction.
