# bw_projects

This is a library to manage subdirectories, so that work on a project can be isolated from other projects. It is designed for use with the [Brightway life cycle assessment](https://brightway.dev/) software framework, but has no dependencies on Brightway and can be used independently.

Project metadata is stored in SQLite using the [Peewee ORM](http://docs.peewee-orm.com/en/latest/). The SQLite file is in the `base_directory`, and project data is stored in subdirectories. By default, [appdirs](https://github.com/ActiveState/appdirs) is used to create the `base_directory`, though this can be overridden.

Example usage:

```python

from bw_projects import projects
# Creates `projects.base_directory` if needed

projects.current
>>> None

projects.set_current("example")
# Create SQLite database row with metadata, relative filepath, and project name
# Creates `projects.base_directory` / `example` if needed

projects.current
>>> "example"

projects.dir
>>> pathlib.Path(`projects.base_directory` / `example`)
```

## Installation

Via pip or conda (`conda-forge` channel).

Depends on:

* [appdirs](https://github.com/ActiveState/appdirs)
* [peewee](http://docs.peewee-orm.com/en/latest/)

## Usage

### The `projects` class instance

`bw_projects` will always create an instance of `ProjectManager`, and make it available as as `projects`. This means that the `base_directory` is always created and populated with a SQLite database file.

You can also create your own instances of `ProjectManager`.

### Specifying the `base_directory`

`bw_projects` will use the following, in order:

* If you instantiate `ProjectManager` manually (instead of importing `projects`), you can pass a folder path.
* The environment variable `BRIGHTWAY_DIR`. This directory is created if it doesn't yet exist.
* The folder directory created by `appdirs.user_data_dir("bw_projects", "projects")` (OS-specific)

### Project management

Create a project and switch to it:

	`projects.set_current("<project_name>")`

Create a project without switching:

	`projects.create_project("<project_name>")`

Iterate over projects:

```python

for p in projects:
	print(p.name, p.relative_path, p.metadata)
```

Delete a project from SQLite file **without deleting the directory**:

	`projects.delete_project("<project_name>")`

Delete the current project from SQLite file **without deleting the directory**:

```python

projects.delete_project()
projects.current 
>>> None
```

Delete a project from SQLite file **and delete the directory**:

	`projects.delete_project("<project_name>", delete_dir=True)`

## Testing

In testing, you want an isolated base directory. You can manually instantiate `ProjectManager` with a temporary directory, or use `projects._use_temp_directory()`.

You can use `projects._is_temp_dir` to test if you are in a test temporary directory.

## Callbacks

To execute additional code when a project is created, activated, deactivated, or deleted, you can register callbacks. Each callback should be a callable object which takes the following positional arguments in order:

* name: str
* relative_path: pathlib.Path instance
* base_directory: pathlib.Path instance
* metadata: dict

Callbacks can be registered by appending them to `projects.callbacks.create`, `projects.callbacks.activate`, `projects.callbacks.deactivate`, and `projects.callbacks.delete`.

Here is an example:

```python
from bw_projects import projects

def switcheroo(name, *args):
	print(f"Hi {name}!")

projects.callbacks.activate.append(switcheroo)

projects.set_current("mom")
>>> "Hi mom!"
```

## Changes versus `brightway2`

No project is selected automatically at startup (i.e. `default` is not created and selected automatically). `projects.current` can be `None`.

The project directory is stored in SQLite instead being generated by a function.

