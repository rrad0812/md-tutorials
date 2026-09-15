# Managing Python Projects With **uv**

The **uv** tool is a high-speed package and project manager for Python. It’s written in Rust and designed to streamline your workflow. It offers fast dependency installation and integrates various functionalities into a single tool.

With **uv**, you can:

- install and manage multiple Python versions,
- create virtual environments,
- efficiently handle project dependencies,
- reproduce working environments,
- and even build and publish a project.

These capabilities make **uv** an all-in-one tool for Python project management.

By the end of this tutorial, you’ll understand that:

- **uv** is a Python package and project manager that integrates multiple functionalities into one tool, offering a comprehensive solution for managing Python projects.
- **uv** is used for:
  - fast dependency installation,
  - virtual environment management,
  - Python version management, and
  - project initialization,
  - enhancing productivity and
  - efficiency.
- **uv** can build and publish Python packages to package repositories like PyPI, supporting a streamlined process from development to distribution.
- **uv** automatically handles virtual environments, creating and managing them as needed to ensure clean and isolated project dependencies.

To dive deeper into managing your Python projects efficiently with **uv**, you should have:

- a basic understanding of using Python virtual environments,
- setting up pyproject.toml files for projects, and
- building distributable packages for a project.

## Getting to Know **uv** for Python

Recently, a few exciting tools built with the Rust programming language have appeared in the Python tooling ecosystem. **Ruff** linter and code formatter for Python — is a well-known and popular example of one of these tools.

In this tutorial, you’ll explore another cool tool made with Rust for Python. You’ll get to know **uv**, an extremely fast Python package and project manager.

The main idea behind these tools is to accelerate your Python workflow by speeding up your project management actions. For example, **Ruff** is 10 to 100 times faster than linters like **Flake8** and code formatters like **Black**. Similarly, for package installation, **uv** is 10 to 100 times faster than **pip**.

Pick a few packages below and start the race to compare how long **uv** and **pip** take to install them. The times are illustrative rather than a strict benchmark:

Additionally, **uv** integrates into one tool most of the functionality provided by tools like **pip**, **pip-tools**, **pipx**, **poetry**, **pyenv**, **twine**, **virtualenv**, and more. Therefore, **uv** is an all-in-one solution.

Here’s a quick list of key **uv** features for managing your Python projects:

- **Fast dependency installation**: Installs dependencies really fast, which is especially useful for large dependency trees.
- **Virtual environment management**: Automatically creates and manages virtual environments.
- **Python version management**: Allows the installation and management of multiple Python versions.
- **Project initialization**: Scaffolds a full Python project, including the root directory, Git repository, virtual environment, pyproject.toml, README, and more.
- **Dependency management**: Installs, updates, removes, and locks direct and transitive dependencies, which allows for environment reproducibility.
- **Package builds and publication management**: Allows you to build and publish packages to package repositories like the Python Package Index (PyPI).
- **Developer tooling support**: Installs and lets you run development tools, such as **pytest**, **Black**, and **Ruff**.

Apart from these features, **uv** is a standalone binary that allows for a smooth installation and quick upgrades. You don’t need to have Python installed on your system to install **uv**.

So, with this quick summary of **uv** and its main features, you’re ready to install this tool on your system. That’s what you’ll do in the following section. Additionally, you’ll learn how to update your **uv** installation.

## Installing **uv** to Manage Python Projects

The first step in using any tool is to install it on your operating system. To install **uv**, you have several options.

- The quickest one would be to use the standalone installer.
- Another friendly option is to install **uv** from PyPI using other tools like **pipx** or **pip**.

In the official **uv** installation guide, you’ll find several other installation options. For example, you can use tools like **Homebrew** and **Cargo**, depending on your current platform and operating system. However, in this tutorial, you’ll only explore

- the standalone installer and
- the PyPI options.

### Using the Standalone Installer

The **uv** project provides a standalone installer that you can use to download and install the tool in your system. Below are the relevant commands for the three main operating systems:

- Windows
- Linux + macOS

```sh
curl -LsSf https://astral.sh/uv/install.sh | sh
```

If you don’t have curl installed on your system, then you can use wget as shown below:

```sh
wget -qO- https://astral.sh/uv/install.sh | sh
```

These commands will download and install the latest binary of **uv** in your system. If you’d like to install a specific version of the tool instead of the latest, then you can add the version number to the download URL right after the **uv**/ part:

- Windows
- Linux + macOS

```sh
https://astral.sh/uv/0.6.12/install.sh
```

With this addition to the download URL, you request the installation of **uv** version 0.6.12. Once you’ve downloaded and installed **uv** with the appropriate command, you can verify the installation by running the following command:

```sh
**uv** --version
uv 0.6.12 (e4e03833f 2025-04-02)
```

Now you’re all set up with **uv**. In this case, you’re using version 0.6.12. Of course, your specific version may differ slightly depending on when you read this tutorial.

You can run the **uv --help** command at any time to display the help page, which provides detailed information about each command and option the tool offers. Go ahead and give it a try!

### Installing From PyPI

You can also install **uv** from the Python Package Index (PyPI) using **pipx**. As with the standalone installer, the **uv** command-line-interface (CLI) will be available globally on your system.

To install **uv** using **pipx**, run the following command:

```sh
pipx install uv
```

Note that for this command to work, you need **pipx** to be available in your environment. This involves an extra step. If you don’t want to install an additional app, then you can use **pip**, which is typically shipped by default with most Python distributions.

However, to have system-wide access to **uv**, you’ll have to install it in your Python system installation. This isn’t recommended because it could clutter your system with packages that you may not use.

### Upgrading to the Latest **uv** Version

As you know, time flies. The **uv** project is currently under active development, which means that new versions are released regularly.

If you’ve installed **uv** using the standalone installer and would like to be up to date with the latest version, then you can run the following command:

```sh
**uv** self update
info: Checking for updates...
success: Upgraded **uv** from v0.6.10 to v0.6.12!
https://github.com/astral-sh/uv/releases/tag/0.6.12
```

The **uv** self update command checks whether a new version is available. If that’s the case, the command downloads the new package and installs it for you.

If you’ve installed **uv** with **pipx** and want to upgrade it, then you can run the command below:

```sh
pipx upgrade **uv**
```

This command allows you to upgrade your **uv** installation to the latest version if you used pipx.

## Handling Python Projects With **uv**

Now comes the fun part—the part where you create and manage a Python project with **uv**. In this section, you’ll explore the **uv** features and commands that allow you to do this quickly and efficiently.

For this section and the rest of the tutorial, you’ll use a sample CLI application that retrieves cat information from the Cat API.

### Creating a Python Project

To create and initialize a Python project with **uv**, navigate to the directory where you want to store the project. Once there, you can run the following command to create and initialize the project:

$ **uv** init rpcats

Note that the project name is up to you. In this example, you’ll use rpcats to denote that this is a Real Python project for retrieving and displaying information about cats.

The command above creates the following directory structure under the rpcats/ folder:

rpcats/
│
├── .git/
│
├── .gitignore
├── .python-version
├── README.md
├── main.py
└── pyproject.toml

This is cool! First, you have the .git/ directory, which is the Git repository for your project. The .gitignore file allows you to define the files and folders you want to skip in your version control workflow. By default, **uv** automatically populates this file with sensible entries for most Python projects, ensuring that unwanted files are excluded from version control.

The .python-version file contains the default Python version for the current project. This file tells **uv** which Python version to use when creating a dedicated virtual environment for the project. Next, you have an empty README.md file that you can use to provide basic documentation for your project.

The main.py file is a placeholder Python file that initially contains the following code:

def main():
    print("Hello from rpcats!")

if __name__ == "__main__":
    main()

In this case, you have a main() function that prints a message to the screen. At the bottom of the file, you have the well-known if __name__ == "__main__" idiom, which is common practice in executable files like this one.

Finally, you have the pyproject.toml file with the following initial content:

[project]
name = "rpcats"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.13"
dependencies = []

This file contains basic configuration key-value pairs under the [project] table. You can update the value of any key to meet your requirements. For example, you can change the description to something like the following:

[project]
name = "rpcats"
version = "0.1.0"
description = "Display cat information for the specified breed."
readme = "README.md"
requires-python = ">=3.13"
dependencies = []

Now, your project has a concrete description. You could also change the project’s version, the required Python version, and so on.

Note: If you want to start managing an existing project with **uv**, then navigate to the project directory and run the following command:

$ **uv** init

This command will create the **uv** project structure for you. It won’t overwrite the main.py file if you have one, but it’ll create the file if it’s missing. It neither modifies your Git repository nor your README.md file.

However, this command won’t work if you already have a pyproject.toml file in place. If that’s the case, then you can move the file to another location and run the **uv** init command. Finally, you can update the new file with any relevant configuration from your old pyproject.toml.

That’s it! You’ve created your first project with **uv**. Now you can run the project’s entry-point script, main.py, to check that everything’s working correctly.
Remove ads
Running the Project’s Entry-Point Script

Once you’ve created the project, you can use **uv** to run the entry-point script, which is the main.py file by default. To run the script, go ahead and execute the following command:

$ **uv** run main.py
Using CPython 3.13.2
Creating virtual environment at: .venv
Hello from rpcats!

The first time you run this command, it’ll display the Python version you’re currently using for the project. It’ll also inform you that **uv** has created a dedicated Python virtual environment for the project. The final output line shows the message that results from calling the main() function defined in main.py.

Note: You can change the entry-point script’s name and location at any time. Naming it main.py isn’t mandatory.

If you check the content of your project’s root directory again, then you’ll find a .venv folder containing the dedicated virtual environment. You’ll also find a new file called **uv**.lock, which is a cross-platform lockfile that contains information about your project’s dependencies. This file ensures that other developers can quickly and exactly reproduce your working environment. This enhances team collaboration and code contributions.

Unlike pyproject.toml, which usually specifies the direct dependencies of your project, the **uv**.lock file contains the exact versions of any dependency installed in the project environment.

The **uv**.lock file uses the TOML format. You should add this file to version control. However, you shouldn’t edit this file manually. Instead, you can let **uv** take care of it. You’ll learn more about this file in a moment.
Using **uv** for Dependency Management

When it comes to managing your project’s dependencies, **uv** makes your life easier with a clean workflow. This workflow allows you to lock your project’s dependencies so that other developers can reproduce your environment exactly and contribute to your code without much setup effort.

In the following sections, you’ll learn the basic **uv** commands that you should use to install, update, and remove external packages and libraries. You’ll also learn how these commands create and update the corresponding configuration files to ensure reproducibility.
Adding and Installing Dependencies

Now, go back to the beginning of the Handling Python Projects With **uv** section, expand the collapsible block, and copy the code for the sample CLI app. Paste the code into the main.py file in your project’s root folder.

Once you have the code in place, you might be tempted to run the app with a command like the following:

$ **uv** run main.py Persian
Traceback (most recent call last):
    ...
ModuleNotFoundError: No module named 'requests'

The app fails with a ModuleNotFoundError exception. Why? Because you haven’t installed the required dependencies in your working environment.

In your cat app example, you need the Requests library, which is a popular library for making HTTP requests in Python. You use this library to retrieve cat information from the Cat API.

Go ahead and run the following command to make **uv** add requests:

$ **uv** add requests

This command lets you add the Requests library to your project’s dependencies. It also installs the library in your project’s virtual environment. Once it finishes running, go ahead and check the content of your pyproject.toml file:

[project]
name = "rpcats"
version = "0.1.0"
description = "Display cat information for the specified breed."
readme = "README.md"
requires-python = ">=3.13"
dependencies = [
    "requests>=2.32.3",
]

Notice how **uv** add automatically updates your list of dependencies by adding requests to the pyproject.toml file. In this case, the installed version should be greater or equal to 2.32.3.

Note: If you’re working on an existing project and want to migrate from a requirements.txt file to using **uv**, then you can run the following command:

$ **uv** add -r requirements.txt

This command imports the dependencies declared in your existing requirements.txt file into the **uv** infrastructure.

Internally, **uv** add also updates the **uv**.lock file with version information for the following:

    Direct dependencies: Packages that your project depends on directly. For example, your cat app depends on requests.
    Transitive dependencies: Packages that support your project’s direct dependencies. For example, the Requests library depends on urllib3, and you get it installed as a transitive dependency.

You don’t have to worry about the **uv**.lock content and shouldn’t edit it yourself. Instead, let **uv** be in charge of it. However, you must version-control it.

As you can conclude, **uv** add does the dependency management work for you. It installs the dependencies, edits the pyproject.toml file if necessary, and keeps the **uv**.lock file up-to-date. This ensures that other developers can reproduce your working environment.

Note: Even though **uv** has an alternative interface that mimics pip, you shouldn’t use it to install dependencies because these commands won’t update the **uv**.lock file or pyproject.toml automatically like **uv** add does.

Would you like to give your cat app another try? Go ahead and run it:

$ **uv** run main.py Persian

-----------Persian------------
Origin: Iran (Persia)
Temperament: Affectionate, Loyal, Sedate, Quiet
Life Span: 14 - 15 years
Weight: 9 - 14 lbs

Learn more: https://en.wikipedia.org/wiki/Persian_(cat)

Now, your cat app is working correctly. It makes API requests, processes the responses, and displays information for the target breed of cat.
Remove ads
Upgrading and Removing Dependencies

You can also use **uv** to update and remove dependencies. For example, say that some time has passed since you released the latest version of your rpcats app, and you’d like to continue the development by adding new features. However, you realize a new version is available for the Requests library.

In this situation, you can use the following command to upgrade requests:

$ **uv** add --upgrade requests

This command upgrades requests and updates its version information in the **uv**.lock file. This action allows you to work with the library’s latest version and ensures that other developers can do the same.

If you decide at any point that your project doesn’t need a dependency anymore, then you can remove it with the following command:

$ **uv** remove <package_name>

For example, if you’d like your rpcats app to check for multiple cats at once, you might want to use a library like aiohttp, which is capable of making async HTTP requests. In that case, you can remove requests and add aiohttp.

It’s important to note that **uv** remove also removes transitive dependencies and updates the pyproject.toml and **uv**.lock files accordingly.
Managing Development Dependencies

In most development environments, you’ll have dependencies that aren’t required for running the code but are vital for developing it. For example, testing libraries like pytest, code formatters like Ruff, and static type checkers like mypy might be some of these development dependencies.

You can use **uv** to manage these types of dependencies. This time, you’ll use the **uv** add command with the --dev flag. For example, say that you want to write tests for the cats app. To do this, you want to use pytest. Because your project doesn’t need this library to run, you can install it as a development dependency:

$ **uv** add --dev pytest

This command will install pytest in your project’s virtual environment and add the library as a development dependency to your pyproject.toml and **uv**.lock files.

If you check the content of pyproject.toml after running the command above, then you’ll find the following:
Filename: pyproject.toml

[project]
name = "rpcats"
version = "0.1.0"
description = "Display cat information for the specified breed."
readme = "README.md"
requires-python = ">=3.13"
dependencies = [
    "requests>=2.32.3",
]

[dependency-groups]
dev = [
    "pytest>=8.3.5",
]

The highlighted lines show how **uv** automatically updated the file to include pytest in your list of development dependencies. You can also check the content of **uv**.lock if you want to confirm that pytest is there as well.

Now you can write the tests for your projects!
Locking and Syncing the Environment

As you’ve already learned, **uv** uses the **uv**.lock file to lock a project’s dependencies. Locking consists of capturing your project’s specific dependencies in the lockfile. This process makes it possible to reproduce your working environment in all possible configurations, including the Python version and distribution, the operating system, and the architecture.

As a counterpart, syncing is the process of installing the required packages from the lockfile into the project’s development environment.

Both locking and syncing processes are automatically handled by **uv**. For example, when you execute **uv** run, the project is locked and synced before the command is invoked. This behavior ensures that your project’s environment is always up to date.

Now, imagine that you’ve pulled the cat app from a GitHub repository and would like to try it. In this scenario, you’ll only have the source code and the **uv** configuration files. You won’t have a proper Python virtual environment with the required dependencies to run the app.

To reproduce this situation, you can go ahead and remove the .venv/ folder from the project’s root directory. Then, let **uv** do the hard work for you:

$ **uv** run main.py "Scottish Fold"
Using CPython 3.13.2
Creating virtual environment at: .venv
Installed 9 packages in 53ms

--------Scottish Fold---------
Origin: United Kingdom
Temperament: Affectionate, Intelligent, Loyal, Playful, Social, Sweet, Loving
Life Span: 11 - 14 years
Weight: 5 - 11 lbs

Learn more: https://en.wikipedia.org/wiki/Scottish_Fold

As you can see, before trying to execute the app, **uv** creates a dedicated virtual environment at the .venv/ directory. Then, it installs the dependencies and finally runs the app.

To make sure that **uv** has reproduced the environment correctly, you can run the following command:

$ **uv** pip list
Package            Version
------------------ ---------
certifi            2025.1.31
charset-normalizer 3.4.1
idna               3.10
iniconfig          2.1.0
packaging          24.2
pluggy             1.5.0
pytest             8.3.5
requests           2.32.3
urllib3            2.3.0

This is the list of packages installed in the project’s virtual environment and their specific versions. You can compare this list with the content of the **uv**.lock file. The packages and versions will coincide, which ensures the exact reproduction of the original development environment.

Note: In the example above, you used the pip interface of **uv**. In this case, that’s perfectly valid because you aren’t changing the environment but retrieving information from it.

Finally, you can explicitly lock and sync your project with the following commands:

$ **uv** lock
$ **uv** sync

These commands are helpful if you encounter issues with running the project and want to ensure that you’re using the correct version of each dependency.
Remove ads
Building and Publishing Packages

Now, say that your cat project is polished and ready to release a new version. You can also use **uv** to build and publish the project to a package repository like PyPI. First, you need to set some final options in the project.toml file. Next, you can build a distribution. You’ll learn about both topics in the following sections.
Configuring the Project

Your cat app has a command-line interface (CLI), so you need to explicitly set the project’s entry-point script so that the build system can properly set up the executable file for the app. Go to the pyproject.toml file and add the following:
Filename: pyproject.toml

[project]
name = "rpcats"
version = "0.1.0"
description = "Display cat information for the specified breed."
readme = "README.md"
requires-python = ">=3.13"
dependencies = [
    "requests>=2.32.3",
]

[dependency-groups]
dev = [
    "pytest>=8.3.5",
]

[project.scripts]
rpcats = "main:main"

With these two additional lines, you ensure that when someone installs your app, they can run the rpcats command from their terminal to execute the code from the main() function, which is stored in the main.py file.

Next, you need to define a build system. In this tutorial, you’ll use Setuptools as the build system. Go ahead and add the following lines to the end of your pyproject.toml file:
Filename: pyproject.toml

[project]
name = "rpcats"
version = "0.1.0"
description = "Display cat information for the specified breed."
readme = "README.md"
requires-python = ">=3.13"
dependencies = [
    "requests>=2.32.3",
]

[dependency-groups]
dev = [
    "pytest>=8.3.5",
]

[project.scripts]
rpcats = "main:main"

[build-system]
requires = ["setuptools>=78.1.0", "wheel>=0.45.1"]
build-backend = "setuptools.build_meta"

The first highlighted line specifies that the build system will require setuptools and wheel. The second highlighted line defines the build backend. With these two additions to the project.toml file, you’re ready to build a distribution for your app.
Building a Distribution

You can use the **uv** build command to build source and binary distributions for your Python project. By default, **uv** build will build the project and place the distributions in a dist/ subdirectory under the project’s root:

$ **uv** build
Building source distribution...
running egg_info
creating rpcats.egg-info
...
Successfully built dist/rpcats-0.1.0.tar.gz
Successfully built dist/rpcats-0.1.0-py3-none-any.whl

When you type in the **uv** build command and press Enter, the building process starts. You’ll see a detailed output on your terminal screen. At the end of the output, you’ll get a message telling you that the package has been successfully built.

You also have the following build options:

    **uv** build --sdist
    **uv** build --wheel

The first command builds a source distribution only, while the second command builds a binary distribution. Note that using **uv** build without any flags will build both the source and binary distributions.
Publishing a Distribution

The final step is to publish the built package to a package repository like PyPI so that your users can download and install it on their systems.

Before publishing a package to PyPI, you should make sure that everything works correctly. To this end, you can publish your package to the TestPyPI index, which is a special repository designed for testing purposes. You’ll need to set up an account in TestPyPI and generate an API token. Once you’ve done that, you can manually edit your pyproject.toml file as shown below:

[project]
name = "rpcats"
version = "0.1.0"
description = "Display cat information for the specified breed."
readme = "README.md"
requires-python = ">=3.13"
dependencies = [
    "requests>=2.32.3",
]

[dependency-groups]
dev = [
    "pytest>=8.3.5",
]

[project.scripts]
rpcats = "main:main"

[build-system]
requires = ["setuptools>=78.1.0", "wheel>=0.45.1"]
build-backend = "setuptools.build_meta"

[[tool.uv.index]]
name = "testpypi"
url = "https://test.pypi.org/simple/"
publish-url = "https://test.pypi.org/legacy/"
explicit = true

With these additions, you set up a custom index called testpypi. You must also change the name key to something unique that doesn’t exist in TestPyPI. Otherwise, you won’t be able to upload the package because the name already exists.

Now, you can use **uv** publish with the --index option to upload your app to TestPyPI:

$ **uv** publish --index testpypi --token your_token_here

To run this command, you’ll need to add your personal API token for TestPyPI following the --token option. Once you’ve entered all the information, press Enter to upload the app to TestPyPI. That should be it!

To try the app, you’ll create a new virtual environment in a different directory. So, go ahead and open a terminal window in any other directory on your hard drive. Then, run the following commands:

$ **uv** venv
Using CPython 3.13.2
Creating virtual environment at: .venv
Activate with: source .venv/bin/activate

$ **uv** pip install -i https://test.pypi.org/simple/ rpcats
  × No solution found when resolving dependencies:
  ╰─▶ Because only requests==2.5.4.1 is available and rpcats==0.1.0 depends on
      requests>=2.32.3, we can conclude that rpcats==0.1.0 cannot be used.
      And because only rpcats==0.1.0 is available and you require rpcats, we
      can conclude that your requirements are unsatisfiable.

With the **uv** venv command, you create a fresh virtual environment in your working directory under the .venv folder.

Note: In the example above, you fall back on using the pip interface for **uv** because you’re not managing a project. Instead, you’re just creating a virtual environment and installing a package for testing purposes. While the pip interface is available for use cases like this, it’s not intended for project management.

Next, you try to install rpcats from TestPyPI using the -i option of **uv** pip. The installation fails because **uv** is trying to install a requests version greater than or equal to 2.32.3, as your dependencies state. However, the latest requests version in TestPyPI is 2.5.4.1, which doesn’t fulfill your needs.

To work around this issue, you can install requests from PyPI and then install rpcats as before:

$ **uv** pip install requests

$ **uv** pip install -i https://test.pypi.org/simple/ rpcats

$ **uv** run rpcats Persian

-----------Persian------------
Origin: Iran (Persia)
Temperament: Affectionate, Loyal, Sedate, Quiet
Life Span: 14 - 15 years
Weight: 9 - 14 lbs

Learn more: https://en.wikipedia.org/wiki/Persian_(cat)

Here, you manually install Requests from PyPI. Next, you install rpcats from TestPyPI with the same command as before. Finally, you run rpcats with the **uv** run command to make sure it works properly. Note that you’re not running main.py, but rpcats as a command instead.

Once you’re sure your project works correctly, you can upload it to PyPI’s main index. Again, you need to have an account and an API token.

Note: To keep PyPI unpolluted, you shouldn’t upload rpcats to the index. Remember that this is just a sample project for learning purposes, not a fully functional app.

Once you have the account and the token at hand, run the following command to publish your package:

$ **uv** publish --token your_token_here

This command uses PyPI as its default package index. As with TestPyPI, you need to enter your API token. Once the command is done, your app will be available in PyPI, and you’ll be able to install it like any other Python package.
Remove ads
Conclusion

You’ve learned all about **uv**, a fast, Rust-based package and project manager for Python. You’ve explored its features for creating projects, setting up virtual environments, managing dependencies, building and publishing projects, and more.

Understanding how to manage Python projects effectively is crucial for any Python developer. With **uv**, you have an all-in-one solution that speeds up your workflow and simplifies project management, making it an invaluable tool for both beginners and experienced developers.

In this tutorial, you’ve learned how to:

    Install **uv** on your operating system
    Create and manage Python projects with **uv**
    Handle dependencies efficiently with **uv** commands
    Build and publish Python packages to PyPI or private indexes
    Set up developer tools within **uv** for a streamlined workflow

With these skills, you’ll manage your Python projects more efficiently, ensuring smooth development and deployment processes. You can use **uv** to handle project dependencies, automate environment setup, and improve collaboration across your development team.