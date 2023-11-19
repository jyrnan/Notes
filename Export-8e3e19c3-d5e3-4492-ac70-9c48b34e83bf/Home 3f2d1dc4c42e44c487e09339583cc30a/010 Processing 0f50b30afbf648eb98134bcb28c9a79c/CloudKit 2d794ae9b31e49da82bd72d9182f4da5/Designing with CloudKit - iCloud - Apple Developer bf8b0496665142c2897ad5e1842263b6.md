# Designing with CloudKit - iCloud - Apple Developer

CloudKit is designed for manageability, flexibility, and power. By organizing apps in containers, CloudKit ensures each app is siloed so its data won’t get entangled with other apps. Specialized databases and zones also let you easily separate app information by access type or function. And together with efficient syncing and sharing capabilities, CloudKit provides a comprehensive feature set that lets you develop powerful cloud apps effortlessly.

CloudKit旨在实现可管理性、灵活性和强大性。 通过在容器中组织应用程序，CloudKit确保每个应用程序都被隔离，以便其数据不会与其他应用程序混淆。 专用数据库和区域也可以轻松地根据访问类型或功能将应用程序信息分开。 与高效的同步和共享功能一起，CloudKit提供了一组全面的功能集，可让您轻松开发强大的云应用程序。

### Containers

CloudKit organizes data using logical spaces called containers. A single CloudKit app typically uses a single container for all of its production data, although you can configure a single app to use multiple containers and multiple apps can use a single container. Each container represents an app’s storage in CloudKit and silos your app’s data. You manage the app’s schema within its container.

### Databases

Containers can include different types of databases: public, private, and shared. Each container has a single public database that is shared among all users of the app. Individual app users also have exclusive access to their own private database, as well as a shared database that allows them to share information with other users (see below).

Through selective use of these databases, you and your users can control data access without having to implement complex Access Control List (ACL) models. If you want to expose data to many of the app’s users, for example, you can easily configure the app's public database to provide this functionality. If users want to keep data available only to themselves, they can publish it in their private database. To explicitly share some of that data with other users, they can also invite specific users to see and edit that data in a shared database.

### Zones

CloudKit’s databases contain zones, which are logical separations of data records. A public database has only one zone, called the default zone. Each private database also has a default zone, but the database can also be separated into multiple custom zones. Shared databases do not have a default zone (see below).

When a record is saved to a public or private database, it's placed by default in that database’s default zone. However, you can choose to place records in custom zones within your private database. Custom zones are useful for grouping related records together for specific purposes. For example, if you have different kinds of data in your private database that need to be updated at different rates on devices, you could group different sets of that data into different zones based on type and sync any changes on a zone-by-zone basis. This lets you avoid syncing all zones at once, eliminating unnecessary data transfers.

### Environments

CloudKit provides separate development and production environments for your projects and apps. When your project is still in the development stage, you store your container’s schema and records in the sandbox environment, which allows you to test or experiment with your database schema before you make it available to users. Only members of your development team have access to the sandbox environment.

After you finish developing your schema, you can move it into production through a process called promotion. When promoted, the schema can only be modified in a forward-compatible manner. This means you can’t delete entity types. You can add fields in an entity, but you can’t modify or remove them. Because you typically don’t have control over the frequency of client updates, this methodology helps ensure continuity between older clients and schema changes. When you’re ready to distribute your app for testing, migrate the development schema to the production environment, copying over its record types, fields, and indexes. Apps sold on the App Store can access only the production environment.

CloudKit uses role-based access control (RBAC) to manage privileges and control access to data in public databases (private databases are private). With CloudKit, you set permission levels for a role, then assign the role to a given record type.

Permissions include read, write, and/or create, and are additive. This means read permissions only allow reading of records, write permissions allow reading of and writing to records, and create permissions allow reading and writing to a record as well as creating new records.

Roles include World, Authenticated, and Creator. The World role includes all iCloud users, whether or not they are authenticated. The Authenticated role includes all iCloud users who are signed in and have been authenticated. The Creator role includes only the authenticated user who has created a record.

### Connectivity

CloudKit provides strongly consistent storage and makes it easy to share that data across a user’s devices or among multiple users. Because the data is shared, your app also needs to keep local records synchronized. Its native framework lets you store your app’s data in CloudKit containers, so a user can access it on multiple devices.