# Upgrade

## 2.0.0

### Deprecate usage of fos rest routing

We are no longer considering the [fos rest routing](https://github.com/handcraftedinthealps/RestRoutingBundle) as a best practice.
All bundles should use the Symfony routing system instead.

Inside your `config/routes/sulu_website.yaml` and `config/routes/sulu_admin.yaml` you can remove the fos rest routing configuration.
First, remove all instances of `type: rest` and also replace `.yml` with `.yaml`:

```diff
# config/routes/sulu_website.yaml`
 sulu_comments:
-    type: rest
-    resource: "@SuluCommentBundle/Resources/config/routing_website.yml"
+    resource: "@SuluCommentBundle/Resources/config/routing_website.yaml"
```

```diff
# config/routes/sulu_admin.yaml`
 sulu_comment_api:
-    type: rest
-    resource: "@SuluCommentBundle/Resources/config/routing_api.yml"
+    resource: "@SuluCommentBundle/Resources/config/routing_api.yaml"
     prefix: /admin/api
```
### Rename comment_count property to commentCount

The `comment_count` property of serialized threads was renamed to `commentCount`.

### Add nested tree to comments

Comments can now be nested - therefor the database schema has changed and can be updated by:

```sql
ALTER TABLE com_comment ADD lft INT NOT NULL, ADD rgt INT NOT NULL, ADD depth INT NOT NULL, ADD idCommentsParent INT DEFAULT NULL;
ALTER TABLE com_comment ADD CONSTRAINT FK_AA6F14A324308710 FOREIGN KEY (idCommentsParent) REFERENCES com_comment (id) ON DELETE CASCADE;
CREATE INDEX IDX_AA6F14A324308710 ON com_comment (idCommentsParent);
```

Use following configuration to disable the nested comments by default:

```yaml
sulu_comment:
    nested_comments: false
``` 

### Type-Hints

We have added type-hints to the whole codebase. Therefor the function parameter and returns validation is stricter
than before.

Additionally we have remove the possibility to pass a single ID to the following functions. If you want to delete a
single entity you have to pass an array with a single id.

* CommentManagerInterface::delete
* CommentManagerInterface::deleteThreads

### Index length of threads type/entityId

In order to allow `utf8mb4` it was neccesary to down size the length of the fields `type` and `entityId` within the
Thread. The following SQL upgrades your database schema. But be sure that the types you are not longer than 64
characters.

```sql
ALTER TABLE com_threads CHANGE type type VARCHAR(64) NOT NULL, CHANGE entityId entityId VARCHAR(64) NOT NULL;
```
