# Python Testing for Databricks

<!-- TOC -->
* [Python Testing for Databricks](#python-testing-for-databricks)
  * [PyTest Fixtures](#pytest-fixtures)
  * [Logging](#logging)
    * [`debug_env_name` fixture](#debug_env_name-fixture)
    * [`debug_env` fixture](#debug_env-fixture)
    * [`env_or_skip` fixture](#env_or_skip-fixture)
    * [`ws` fixture](#ws-fixture)
    * [`make_random` fixture](#make_random-fixture)
    * [`make_instance_pool` fixture](#make_instance_pool-fixture)
    * [`make_instance_pool_permissions` fixture](#make_instance_pool_permissions-fixture)
    * [`make_job` fixture](#make_job-fixture)
    * [`make_job_permissions` fixture](#make_job_permissions-fixture)
    * [`make_cluster` fixture](#make_cluster-fixture)
    * [`make_cluster_permissions` fixture](#make_cluster_permissions-fixture)
    * [`make_cluster_policy` fixture](#make_cluster_policy-fixture)
    * [`make_cluster_policy_permissions` fixture](#make_cluster_policy_permissions-fixture)
    * [`make_group` fixture](#make_group-fixture)
    * [`make_user` fixture](#make_user-fixture)
    * [`make_pipeline_permissions` fixture](#make_pipeline_permissions-fixture)
    * [`make_notebook` fixture](#make_notebook-fixture)
    * [`make_notebook_permissions` fixture](#make_notebook_permissions-fixture)
    * [`make_directory` fixture](#make_directory-fixture)
    * [`make_directory_permissions` fixture](#make_directory_permissions-fixture)
    * [`make_repo` fixture](#make_repo-fixture)
    * [`make_repo_permissions` fixture](#make_repo_permissions-fixture)
    * [`make_workspace_file_permissions` fixture](#make_workspace_file_permissions-fixture)
    * [`make_workspace_file_path_permissions` fixture](#make_workspace_file_path_permissions-fixture)
    * [`make_secret_scope` fixture](#make_secret_scope-fixture)
    * [`make_secret_scope_acl` fixture](#make_secret_scope_acl-fixture)
    * [`make_authorization_permissions` fixture](#make_authorization_permissions-fixture)
    * [`make_udf` fixture](#make_udf-fixture)
    * [`make_catalog` fixture](#make_catalog-fixture)
    * [`make_schema` fixture](#make_schema-fixture)
    * [`make_table` fixture](#make_table-fixture)
    * [`product_info` fixture](#product_info-fixture)
    * [`sql_backend` fixture](#sql_backend-fixture)
    * [`sql_exec` fixture](#sql_exec-fixture)
    * [`sql_fetch_all` fixture](#sql_fetch_all-fixture)
    * [`make_experiment` fixture](#make_experiment-fixture)
    * [`make_experiment_permissions` fixture](#make_experiment_permissions-fixture)
    * [`make_warehouse_permissions` fixture](#make_warehouse_permissions-fixture)
    * [`make_lakeview_dashboard_permissions` fixture](#make_lakeview_dashboard_permissions-fixture)
    * [`workspace_library` fixture](#workspace_library-fixture)
    * [`log_workspace_link` fixture](#log_workspace_link-fixture)
    * [`make_dashboard_permissions` fixture](#make_dashboard_permissions-fixture)
    * [`make_alert_permissions` fixture](#make_alert_permissions-fixture)
    * [`make_query_permissions` fixture](#make_query_permissions-fixture)
    * [`make_registered_model_permissions` fixture](#make_registered_model_permissions-fixture)
    * [`make_serving_endpoint_permissions` fixture](#make_serving_endpoint_permissions-fixture)
    * [`make_feature_table_permissions` fixture](#make_feature_table_permissions-fixture)
* [Project Support](#project-support)
<!-- TOC -->

## PyTest Fixtures

Pytest fixtures are a powerful way to manage test setup and teardown in Python.

[[back to top](#python-testing-for-databricks)]

## Logging

This library is built on years of debugging integration tests for Databricks and its ecosystem. 

That's why it comes with a built-in logger that traces creation and deletion of dummy entities through links in 
the Databricks Workspace UI. If you run the following code:

```python
def test_new_user(make_user, ws):
    new_user = make_user()
    home_dir = ws.workspace.get_status(f"/Users/{new_user.user_name}")
    assert home_dir.object_type == ObjectType.DIRECTORY
```

You will see the following output, where the first line is clickable and will take you to the user's profile in the Databricks Workspace UI:

```text
12:30:53  INFO [d.l.p.fixtures.baseline] Created dummy-xwuq-...@example.com: https://.....azuredatabricks.net/#settings/workspace/identity-and-access/users/735...
12:30:53 DEBUG [d.l.p.fixtures.baseline] added workspace user fixture: User(active=True, display_name='dummy-xwuq-...@example.com', ...)
12:30:58 DEBUG [d.l.p.fixtures.baseline] clearing 1 workspace user fixtures
12:30:58 DEBUG [d.l.p.fixtures.baseline] removing workspace user fixture: User(active=True, display_name='dummy-xwuq-...@example.com', ...)
```

You may need to add the following to your `conftest.py` file to enable this:

```python
import logging

from databricks.labs.blueprint.logger import install_logger

install_logger()

logging.getLogger('databricks.labs.pytester').setLevel(logging.DEBUG)
```

[[back to top](#python-testing-for-databricks)]

<!-- FIXTURES -->
### `debug_env_name` fixture
Specify the name of the debug environment. By default, it is set to `.env`,
which will try to find a [file named `.env`](https://www.dotenv.org/docs/security/env)
in any of the parent directories of the current working directory and load
the environment variables from it via the [`debug_env` fixture](#debug_env-fixture).

Alternatively, if you are concerned of the
[risk of `.env` files getting checked into version control](https://thehackernews.com/2024/08/attackers-exploit-public-env-files-to.html),
we recommend using the `~/.databricks/debug-env.json` file to store different sets of environment variables.
The file cannot be checked into version control by design, because it is stored in the user's home directory.

This file is used for local debugging and integration tests in IDEs like PyCharm, VSCode, and IntelliJ IDEA
while developing Databricks Platform Automation Stack, which includes Databricks SDKs for Python, Go, and Java,
as well as Databricks Terraform Provider and Databricks CLI. This file enables multi-environment and multi-cloud
testing with a single set of integration tests.

The file is typically structured as follows:

```shell
$ cat ~/.databricks/debug-env.json
{
   "ws": {
     "CLOUD_ENV": "azure",
     "DATABRICKS_HOST": "....azuredatabricks.net",
     "DATABRICKS_CLUSTER_ID": "0708-200540-...",
     "DATABRICKS_WAREHOUSE_ID": "33aef...",
        ...
   },
   "acc": {
     "CLOUD_ENV": "aws",
     "DATABRICKS_HOST": "accounts.cloud.databricks.net",
     "DATABRICKS_CLIENT_ID": "....",
     "DATABRICKS_CLIENT_SECRET": "....",
     ...
   }
}
```

And you can load it in your `conftest.py` file as follows:

```python
@pytest.fixture
def debug_env_name():
    return "ws"
```

This will load the `ws` environment from the `~/.databricks/debug-env.json` file.

If any of the environment variables are not found, [`env_or_skip` fixture](#env_or_skip-fixture)
will gracefully skip the execution of tests.

See also [`debug_env`](#debug_env-fixture).


[[back to top](#python-testing-for-databricks)]

### `debug_env` fixture
Loads environment variables specified in [`debug_env_name` fixture](#debug_env_name-fixture) from a file
for local debugging in IDEs, otherwise allowing the tests to run with the default environment variables
specified in the CI/CD pipeline.

See also [`env_or_skip`](#env_or_skip-fixture), [`ws`](#ws-fixture), [`debug_env_name`](#debug_env_name-fixture).


[[back to top](#python-testing-for-databricks)]

### `env_or_skip` fixture
Fixture to get environment variables or skip tests.

It is extremely useful to skip tests if the required environment variables are not set.

In the following example, `test_something` would only run if the environment variable
`SOME_EXTERNAL_SERVICE_TOKEN` is set:

```python
def test_something(env_or_skip):
    token = env_or_skip("SOME_EXTERNAL_SERVICE_TOKEN")
    assert token is not None
```

See also [`make_udf`](#make_udf-fixture), [`sql_backend`](#sql_backend-fixture), [`debug_env`](#debug_env-fixture).


[[back to top](#python-testing-for-databricks)]

### `ws` fixture
Create and provide a Databricks WorkspaceClient object.

This fixture initializes a Databricks WorkspaceClient object, which can be used
to interact with the Databricks workspace API. The created instance of WorkspaceClient
is shared across all test functions within the test session.

See [detailed documentation](https://databricks-sdk-py.readthedocs.io/en/latest/authentication.html) for the list
of environment variables that can be used to authenticate the WorkspaceClient.

In your test functions, include this fixture as an argument to use the WorkspaceClient:

```python
def test_workspace_operations(ws):
    clusters = ws.clusters.list_clusters()
    assert len(clusters) >= 0
```

See also [`log_workspace_link`](#log_workspace_link-fixture), [`make_alert_permissions`](#make_alert_permissions-fixture), [`make_authorization_permissions`](#make_authorization_permissions-fixture), [`make_catalog`](#make_catalog-fixture), [`make_cluster`](#make_cluster-fixture), [`make_cluster_permissions`](#make_cluster_permissions-fixture), [`make_cluster_policy`](#make_cluster_policy-fixture), [`make_cluster_policy_permissions`](#make_cluster_policy_permissions-fixture), [`make_dashboard_permissions`](#make_dashboard_permissions-fixture), [`make_directory`](#make_directory-fixture), [`make_directory_permissions`](#make_directory_permissions-fixture), [`make_experiment`](#make_experiment-fixture), [`make_experiment_permissions`](#make_experiment_permissions-fixture), [`make_feature_table_permissions`](#make_feature_table_permissions-fixture), [`make_group`](#make_group-fixture), [`make_instance_pool`](#make_instance_pool-fixture), [`make_instance_pool_permissions`](#make_instance_pool_permissions-fixture), [`make_job`](#make_job-fixture), [`make_job_permissions`](#make_job_permissions-fixture), [`make_lakeview_dashboard_permissions`](#make_lakeview_dashboard_permissions-fixture), [`make_notebook`](#make_notebook-fixture), [`make_notebook_permissions`](#make_notebook_permissions-fixture), [`make_pipeline_permissions`](#make_pipeline_permissions-fixture), [`make_query_permissions`](#make_query_permissions-fixture), [`make_registered_model_permissions`](#make_registered_model_permissions-fixture), [`make_repo`](#make_repo-fixture), [`make_repo_permissions`](#make_repo_permissions-fixture), [`make_schema`](#make_schema-fixture), [`make_secret_scope`](#make_secret_scope-fixture), [`make_secret_scope_acl`](#make_secret_scope_acl-fixture), [`make_serving_endpoint_permissions`](#make_serving_endpoint_permissions-fixture), [`make_table`](#make_table-fixture), [`make_udf`](#make_udf-fixture), [`make_user`](#make_user-fixture), [`make_warehouse_permissions`](#make_warehouse_permissions-fixture), [`make_workspace_file_path_permissions`](#make_workspace_file_path_permissions-fixture), [`make_workspace_file_permissions`](#make_workspace_file_permissions-fixture), [`sql_backend`](#sql_backend-fixture), [`workspace_library`](#workspace_library-fixture), [`debug_env`](#debug_env-fixture), [`product_info`](#product_info-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_random` fixture
Fixture to generate random strings.

This fixture provides a function to generate random strings of a specified length.
The generated strings are created using a character set consisting of uppercase letters,
lowercase letters, and digits.

To generate a random string with default length of 16 characters:

```python
random_string = make_random()
assert len(random_string) == 16
```

To generate a random string with a specified length:

```python
random_string = make_random(k=8)
assert len(random_string) == 8
```

See also [`make_catalog`](#make_catalog-fixture), [`make_cluster`](#make_cluster-fixture), [`make_cluster_policy`](#make_cluster_policy-fixture), [`make_directory`](#make_directory-fixture), [`make_experiment`](#make_experiment-fixture), [`make_group`](#make_group-fixture), [`make_instance_pool`](#make_instance_pool-fixture), [`make_job`](#make_job-fixture), [`make_notebook`](#make_notebook-fixture), [`make_repo`](#make_repo-fixture), [`make_schema`](#make_schema-fixture), [`make_secret_scope`](#make_secret_scope-fixture), [`make_table`](#make_table-fixture), [`make_udf`](#make_udf-fixture), [`make_user`](#make_user-fixture), [`workspace_library`](#workspace_library-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_instance_pool` fixture
Create a Databricks instance pool and clean it up after the test. Returns a function to create instance pools.
Use `instance_pool_id` attribute from the returned object to get an ID of the pool.

Keyword Arguments:
* `instance_pool_name` (str, optional): The name of the instance pool. If not provided, a random name will be generated.
* `node_type_id` (str, optional): The node type ID of the instance pool. If not provided, a node type with local disk and 16GB memory will be used.
* other arguments are passed to `WorkspaceClient.instance_pools.create` method.

Usage:
```python
def test_instance_pool(make_instance_pool):
    logger.info(f"created {make_instance_pool()}")
```

See also [`ws`](#ws-fixture), [`make_random`](#make_random-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_instance_pool_permissions` fixture
_No description yet._

See also [`ws`](#ws-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_job` fixture
Create a Databricks job and clean it up after the test. Returns a function to create jobs.

Keyword Arguments:
* `notebook_path` (str, optional): The path to the notebook. If not provided, a random notebook will be created.
* `name` (str, optional): The name of the job. If not provided, a random name will be generated.
* `spark_conf` (dict, optional): The Spark configuration of the job.
* `libraries` (list, optional): The list of libraries to install on the job.
* other arguments are passed to `WorkspaceClient.jobs.create` method.

If no task argument is provided, a single task with a notebook task will be created, along with a disposable notebook.
Latest Spark version and a single worker clusters will be used to run this ephemeral job.

Usage:
```python
def test_job(make_job):
    logger.info(f"created {make_job()}")
```

See also [`ws`](#ws-fixture), [`make_random`](#make_random-fixture), [`make_notebook`](#make_notebook-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_job_permissions` fixture
_No description yet._

See also [`ws`](#ws-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_cluster` fixture
Create a Databricks cluster, waits for it to start, and clean it up after the test.
Returns a function to create clusters. You can get `cluster_id` attribute from the returned object.

Keyword Arguments:
* `single_node` (bool, optional): Whether to create a single-node cluster. Defaults to False.
* `cluster_name` (str, optional): The name of the cluster. If not provided, a random name will be generated.
* `spark_version` (str, optional): The Spark version of the cluster. If not provided, the latest version will be used.
* `autotermination_minutes` (int, optional): The number of minutes before the cluster is automatically terminated. Defaults to 10.

Usage:
```python
def test_cluster(make_cluster):
    logger.info(f"created {make_cluster(single_node=True)}")
```

See also [`ws`](#ws-fixture), [`make_random`](#make_random-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_cluster_permissions` fixture
_No description yet._

See also [`ws`](#ws-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_cluster_policy` fixture
Create a Databricks cluster policy and clean it up after the test. Returns a function to create cluster policies,
which returns [`CreatePolicyResponse`](https://databricks-sdk-py.readthedocs.io/en/latest/dbdataclasses/compute.html#databricks.sdk.service.compute.CreatePolicyResponse) instance.

Keyword Arguments:
* `name` (str, optional): The name of the cluster policy. If not provided, a random name will be generated.

Usage:
```python
def test_cluster_policy(make_cluster_policy):
    logger.info(f"created {make_cluster_policy()}")
```

See also [`ws`](#ws-fixture), [`make_random`](#make_random-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_cluster_policy_permissions` fixture
_No description yet._

See also [`ws`](#ws-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_group` fixture
This fixture provides a function to manage Databricks workspace groups. Groups can be created with
specified members and roles, and they will be deleted after the test is complete. Deals with eventual
consistency issues by retrying the creation process for 30 seconds and allowing up to two minutes
for group to be provisioned. Returns an instance of [`Group`](https://databricks-sdk-py.readthedocs.io/en/latest/dbdataclasses/iam.html#databricks.sdk.service.iam.Group).

Keyword arguments:
* `members` (list of strings): A list of user IDs to add to the group.
* `roles` (list of strings): A list of roles to assign to the group.
* `display_name` (str): The display name of the group.
* `wait_for_provisioning` (bool): If `True`, the function will wait for the group to be provisioned.
* `entitlements` (list of strings): A list of entitlements to assign to the group.

The following example creates a group with a single member and independently verifies that the group was created:

```python
def test_new_group(make_group, make_user, ws):
    user = make_user()
    group = make_group(members=[user.id])
    loaded = ws.groups.get(group.id)
    assert group.display_name == loaded.display_name
    assert group.members == loaded.members
```

See also [`ws`](#ws-fixture), [`make_random`](#make_random-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_user` fixture
This fixture returns a function that creates a Databricks workspace user
and removes it after the test is complete. In case of random naming conflicts,
the fixture will retry the creation process for 30 seconds. Returns an instance
of [`User`](https://databricks-sdk-py.readthedocs.io/en/latest/dbdataclasses/iam.html#databricks.sdk.service.iam.User). Usage:

```python
def test_new_user(make_user, ws):
    new_user = make_user()
    home_dir = ws.workspace.get_status(f"/Users/{new_user.user_name}")
    assert home_dir.object_type == ObjectType.DIRECTORY
```

See also [`ws`](#ws-fixture), [`make_random`](#make_random-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_pipeline_permissions` fixture
_No description yet._

See also [`ws`](#ws-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_notebook` fixture
Returns a function to create Databricks Notebooks and clean them up after the test.
The function returns [`os.PathLike` object](https://github.com/databrickslabs/blueprint?tab=readme-ov-file#python-native-pathlibpath-like-interfaces).

Keyword arguments:
* `path` (str, optional): The path of the notebook. Defaults to `dummy-*` notebook in current user's home folder.
* `content` (typing.BinaryIO, optional): The content of the notebook. Defaults to `print(1)`.
* `language` ([`Language`](https://databricks-sdk-py.readthedocs.io/en/latest/dbdataclasses/workspace.html#databricks.sdk.service.workspace.Language), optional): The language of the notebook. Defaults to `Language.PYTHON`.
* `format` ([`ImportFormat`](https://databricks-sdk-py.readthedocs.io/en/latest/dbdataclasses/workspace.html#databricks.sdk.service.workspace.ImportFormat), optional): The format of the notebook. Defaults to `ImportFormat.SOURCE`.
* `overwrite` (bool, optional): Whether to overwrite the notebook if it already exists. Defaults to `False`.

This example creates a notebook and verifies that `print(1)` is in the content:
```python
def test_creates_some_notebook(make_notebook):
    notebook = make_notebook()
    assert "print(1)" in notebook.read_text()
```

See also [`make_job`](#make_job-fixture), [`ws`](#ws-fixture), [`make_random`](#make_random-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_notebook_permissions` fixture
_No description yet._

See also [`ws`](#ws-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_directory` fixture
Returns a function to create Databricks Workspace Folders and clean them up after the test.
The function returns [`os.PathLike` object](https://github.com/databrickslabs/blueprint?tab=readme-ov-file#python-native-pathlibpath-like-interfaces).

Keyword arguments:
* `path` (str, optional): The path of the notebook. Defaults to `dummy-*` folder in current user's home folder.

This example creates a folder and verifies that it contains a notebook:
```python
def test_creates_some_folder_with_a_notebook(make_directory, make_notebook):
    folder = make_directory()
    notebook = make_notebook(path=folder / 'foo.py')
    files = [_.name for _ in folder.iterdir()]
    assert ['foo.py'] == files
    assert notebook.parent == folder
```

See also [`make_experiment`](#make_experiment-fixture), [`ws`](#ws-fixture), [`make_random`](#make_random-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_directory_permissions` fixture
_No description yet._

See also [`ws`](#ws-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_repo` fixture
Returns a function to create Databricks Repos and clean them up after the test.
The function returns a [`RepoInfo`](https://databricks-sdk-py.readthedocs.io/en/latest/dbdataclasses/workspace.html#databricks.sdk.service.workspace.RepoInfo) object.

Keyword arguments:
* `url` (str, optional): The URL of the repository.
* `provider` (str, optional): The provider of the repository.
* `path` (str, optional): The path of the repository. Defaults to `/Repos/{current_user}/sdk-{random}-{purge_suffix}`.

Usage:
```python
def test_repo(make_repo):
    logger.info(f"created {make_repo()}")
```

See also [`ws`](#ws-fixture), [`make_random`](#make_random-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_repo_permissions` fixture
_No description yet._

See also [`ws`](#ws-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_workspace_file_permissions` fixture
_No description yet._

See also [`ws`](#ws-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_workspace_file_path_permissions` fixture
_No description yet._

See also [`ws`](#ws-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_secret_scope` fixture
This fixture provides a function to create secret scopes. The created secret scope will be
deleted after the test is complete. Returns the name of the secret scope.

To create a secret scope and use it within a test function:

```python
def test_secret_scope_creation(make_secret_scope):
    secret_scope_name = make_secret_scope()
    assert secret_scope_name.startswith("dummy-")
```

See also [`ws`](#ws-fixture), [`make_random`](#make_random-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_secret_scope_acl` fixture
This fixture provides a function to manage access control lists (ACLs) for secret scopes.
ACLs define permissions for principals (users or groups) on specific secret scopes.

Arguments:
- `scope`: The name of the secret scope.
- `principal`: The name of the principal (user or group).
- `permission`: The permission level for the principal on the secret scope.

Returns a tuple containing the secret scope name and the principal name.

To manage secret scope ACLs using the make_secret_scope_acl fixture:

```python
from databricks.sdk.service.workspace import AclPermission

def test_secret_scope_acl_management(make_user, make_secret_scope, make_secret_scope_acl):
    scope_name = make_secret_scope()
    principal_name = make_user().display_name
    permission = AclPermission.READ

    acl_info = make_secret_scope_acl(
        scope=scope_name,
        principal=principal_name,
        permission=permission,
    )
    assert acl_info == (scope_name, principal_name)
```

See also [`ws`](#ws-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_authorization_permissions` fixture
_No description yet._

See also [`ws`](#ws-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_udf` fixture
_No description yet._

See also [`ws`](#ws-fixture), [`env_or_skip`](#env_or_skip-fixture), [`sql_backend`](#sql_backend-fixture), [`make_schema`](#make_schema-fixture), [`make_random`](#make_random-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_catalog` fixture
_No description yet._

See also [`ws`](#ws-fixture), [`sql_backend`](#sql_backend-fixture), [`make_random`](#make_random-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_schema` fixture
_No description yet._

See also [`make_table`](#make_table-fixture), [`make_udf`](#make_udf-fixture), [`ws`](#ws-fixture), [`sql_backend`](#sql_backend-fixture), [`make_random`](#make_random-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_table` fixture
_No description yet._

See also [`ws`](#ws-fixture), [`sql_backend`](#sql_backend-fixture), [`make_schema`](#make_schema-fixture), [`make_random`](#make_random-fixture).


[[back to top](#python-testing-for-databricks)]

### `product_info` fixture
_No description yet._

See also [`ws`](#ws-fixture).


[[back to top](#python-testing-for-databricks)]

### `sql_backend` fixture
te and provide a SQL backend for executing statements.

Requires the environment variable `DATABRICKS_WAREHOUSE_ID` to be set.

See also [`make_catalog`](#make_catalog-fixture), [`make_schema`](#make_schema-fixture), [`make_table`](#make_table-fixture), [`make_udf`](#make_udf-fixture), [`sql_exec`](#sql_exec-fixture), [`sql_fetch_all`](#sql_fetch_all-fixture), [`ws`](#ws-fixture), [`env_or_skip`](#env_or_skip-fixture).


[[back to top](#python-testing-for-databricks)]

### `sql_exec` fixture
ute SQL statement and don't return any results.

See also [`sql_backend`](#sql_backend-fixture).


[[back to top](#python-testing-for-databricks)]

### `sql_fetch_all` fixture
h all rows from a SQL statement.

See also [`sql_backend`](#sql_backend-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_experiment` fixture
Returns a function to create Databricks Experiments and clean them up after the test.
The function returns a [`CreateExperimentResponse`](https://databricks-sdk-py.readthedocs.io/en/latest/dbdataclasses/ml.html#databricks.sdk.service.ml.CreateExperimentResponse) object.

Keyword arguments:
* `path` (str, optional): The path of the experiment. Defaults to `dummy-*` experiment in current user's home folder.
* `experiment_name` (str, optional): The name of the experiment. Defaults to `dummy-*`.

Usage:
```python
from databricks.sdk.service.iam import PermissionLevel

def test_experiments(make_group, make_experiment, make_experiment_permissions):
    group = make_group()
    experiment = make_experiment()
    make_experiment_permissions(
        object_id=experiment.experiment_id,
        permission_level=PermissionLevel.CAN_MANAGE,
        group_name=group.display_name,
    )
```

See also [`ws`](#ws-fixture), [`make_random`](#make_random-fixture), [`make_directory`](#make_directory-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_experiment_permissions` fixture
_No description yet._

See also [`ws`](#ws-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_warehouse_permissions` fixture
_No description yet._

See also [`ws`](#ws-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_lakeview_dashboard_permissions` fixture
_No description yet._

See also [`ws`](#ws-fixture).


[[back to top](#python-testing-for-databricks)]

### `workspace_library` fixture
_No description yet._

See also [`ws`](#ws-fixture), [`make_random`](#make_random-fixture).


[[back to top](#python-testing-for-databricks)]

### `log_workspace_link` fixture
rns a function to log a workspace link.

See also [`ws`](#ws-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_dashboard_permissions` fixture
_No description yet._

See also [`ws`](#ws-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_alert_permissions` fixture
_No description yet._

See also [`ws`](#ws-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_query_permissions` fixture
_No description yet._

See also [`ws`](#ws-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_registered_model_permissions` fixture
_No description yet._

See also [`ws`](#ws-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_serving_endpoint_permissions` fixture
_No description yet._

See also [`ws`](#ws-fixture).


[[back to top](#python-testing-for-databricks)]

### `make_feature_table_permissions` fixture
_No description yet._

See also [`ws`](#ws-fixture).


[[back to top](#python-testing-for-databricks)]

<!-- END FIXTURES -->

# Project Support

Please note that this project is provided for your exploration only and is not 
formally supported by Databricks with Service Level Agreements (SLAs). They are 
provided AS-IS, and we do not make any guarantees of any kind. Please do not 
submit a support ticket relating to any issues arising from the use of this project.

Any issues discovered through the use of this project should be filed as GitHub 
[Issues on this repository](https://github.com/databrickslabs/pytester/issues). 
They will be reviewed as time permits, but no formal SLAs for support exist.
