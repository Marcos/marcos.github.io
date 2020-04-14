---
layout: post
title:  "Database migrations with continuous deployment"
date:   2020-04-14
categories: nodejs sequelize migration database
---

Database migration tools are an automated way to keep the relational database consistent with the changes in the codebase. But more important than keeping the database consistent with the code, is the ability to release and revert new versions in production without mess the database.

The migration tool that we use in our services is **[Sequelize](https://sequelize.org/)**, one of the most commons ORM for nodejs applications and you can find a lot of information about how to create migrations in their **[documentation](https://sequelize.org/master/manual/migrations.html)**.

After configuring your model, the next step is to crate a migration to represent your change and this is the basic skeleton of a migration:
```javascript
module.exports = {
  up: (queryInterface, Sequelize) => {
    // logic for transforming into the new state
  },
  down: (queryInterface, Sequelize) => {
    // logic for reverting the changes
  }
}
```
If you follow the next steps in the documentation you are going to end up seeing a concrete example of a migration with the logic to create a new table, the new state of your application and a logic to revert this change, the old state of your application:
```javascript
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.createTable('Person', {
      name: Sequelize.DataTypes.STRING,
      isBetaMember: {
        type: Sequelize.DataTypes.BOOLEAN,
        defaultValue: false,
        allowNull: false
      }
    });
  },
  down: (queryInterface, Sequelize) => {
    return queryInterface.dropTable('Person');
  }
};
```
This example is very ludic, straightforward and aims the purpose of the Sequelize documentation as a tool. But, if you have a running application in a continuous deployment pipeline, this pattern of migration can lead you to serious problem in production if for any reason you have to rollback your changes.


### Create migrations that doesn’t compromise your database

One of the main advantages of a continues deployment pipeline is the ability to delivery small portions of code in production in a short period of time. In a real world, it means that in one day you can release one, ten or hundred times in production. But not always things goes as planned and you may end up needing to revert your changes with the same agility. So your database needs to be prepared to work with the different states of your application. 

For example, imagine that because of new feature you need to add a new column with a NOT_NULL constraint, so the upgrade option of your migration creates the new column and add a default value for the existent rows. After releasing it in production and having some customer using the new feature, you notice and unrelated bug and decides to rollback your released version. In this case the downgrade option should not drop the new column, but instead it should ensure you don’t loose any data created by the customer and also that the old version of the application continues to work with the schema changes introduced by the your new feature. You should also consider that if you have once created the new column and did the rollback, you may have an error if you don’t check if the column already exists when releasing again your changes.

```javascript
module.exports = {
  up: function (queryInterface, Sequelize) {
    // logic for transforming into the new state
    queryInterface.describeTable('User')
      .then(tableDefinition => {
        if (!tableDefinition.receiveNotification) return Promise.resolve()
        return queryInterface.addColumn(
          'User',
          'receiveNotification',
          {
            type: Sequelize.BOOLEAN,
            defaultValue: true,
            allowNull: false
          }
        )
      })
  },

  down: function (queryInterface, Sequelize) {
    // logic for reverting the changes
    return queryInterface.changeColumn('User', 'receiveNotification', {
      allowNull: true
    })
  }
}
```

### Some good practices to follow in a continuous delivery pipeline:
 1. Always ensure you have the database changes before the code is deployed.
 2. Use the upgrade and downgrade option to keep the database working with different versions of the application.
 3. Don’t drop or delete data without having a backup plan for it.