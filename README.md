# Databricks Token Monitoring and Revocation Process

This notebook provides a process for monitoring and revoking Databricks tokens. It allows administrators to monitor all tokens created in the current workspace and revoke tokens associated with a specific user by providing their email address.

## Features

- **Token Monitoring**: Lists all tokens created in the current Databricks workspace.
- **Token Revocation**: Revokes all tokens associated with a specific user by providing their email address.
- **Audit Logging**: Logs token details into a monitoring table for auditing purposes.

## Usage

### Input Parameters

- **User Email**: An optional parameter to specify the email of the user whose tokens need to be revoked. If left empty, the notebook will only monitor tokens without revoking any.

### Steps

1. **Token Monitoring**:
   - The notebook retrieves all tokens created in the current workspace and displays them in a table.
   - It also logs the token details into a monitoring table (`monitor.logging.databricks_tokens`) for auditing purposes.

2. **Token Revocation**:
   - If a user email is provided, the notebook will revoke all tokens associated with that user.
   - The revocation process logs the token IDs and the user email for reference.

### Code Overview

- **Token Retrieval**: The notebook uses the Databricks SDK to list all tokens in the workspace and creates a temporary view for further processing.
- **Workspace ID Retrieval**: The workspace ID is retrieved for auditing purposes.
- **Token Logging**: Token details are logged into a monitoring table.
- **Token Revocation**: Tokens associated with the provided user email are revoked using the Databricks SDK.

### Example

```python
# Input parameter: User email
dbutils.widgets.text("user_email", "", "User Email")
user_email = dbutils.widgets.get("user_email")

# Retrieve and display tokens
from databricks.sdk import WorkspaceClient
w = WorkspaceClient()
spark.createDataFrame([token.as_dict() for token in w.token_management.list()]).createOrReplaceTempView('tokens')
display(spark.sql('select * from tokens order by creation_time'))

# Revoke tokens for the specified user
if user_email:
    for df in spark.sql(f"select * from monitor.logging.databricks_tokens where created_by_username='{user_email}'").collect():
        token_id = df['token_id']
        if token_id:
            w.token_management.delete(token_id=token_id)
            print(f"Token {token_id} revoked for user {user_email}")
