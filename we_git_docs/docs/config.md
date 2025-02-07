# Configuration Files
---
A **configuration file (config file)** is a structured file used to store settings and preferences for an application or system. These files allow applications to remain flexible and user-friendly by enabling customization without modifying the underlying code.

## What is a Config File?

Configuration files are essential for:
- **Storing Preferences**: They remember how users like to interact with an application.
- **Application Setup**: They help applications interact with other systems or environments by providing necessary parameters.

For example, a config file might include details like:
- User's name
- Email address
- Preferred theme (light/dark mode)

---

## Structure of Config Files

Config files generally use **key-value pairs** to represent settings:
- **Key**: An identifier for the setting.
- **Value**: The data or parameter associated with the key.

### Example:
plaintext
username = JohnDoe
email = johndoe@example.com
theme = dark ```

For more complex applications, config files can also store hierarchical data, which organizes settings in a tree-like structure.
<br>
To know about the different [config file formats](https://opensource.com/article/21/6/what-config-files).


## Why Use Config Files?
- Ease of Customization: Users can update settings without modifying the source code.
- Separation of Concerns: Keeps application logic separate from its configuration.
- Flexibility: Developers can easily adjust parameters for testing, deployment, or user preferences.

## Types Of Config Files?
- **System Git** config controls settings for all users and all repositories on your computer.
- **Global Git** config controls settings for the currently logged in user and all his repositories.
- **Local Git** config controls settings for a specific repository.
---
## How to Create a Config File
Steps:
- Identify the keys your file requires
- Choose a Format: Select the appropriate format (INI, JSON, YAML, XML, or .env).
- Write the File
- Create *config parser*

### Indentifying keys
For our project initially we decided to go with two sections namely repository and user.
- Repository
 * defaultBranch
- User
 * name 
 * email
 * username

### Choosing a format
You can choose a format depending on your project and language.
We decided to go forward with INI files.

#### Why INI for us?
* Exploring new formats.
* Simple and widely understood.
* Best for very basic key-value pairs.
* Lightweight and straightforward.
* Easy to parse with libraries like ```ini4j``` for Scala
* Limited support for nested structures.
* Less suitable for complex configurations.
* Easy to parse with package like ```gopkg.in/ini.vi```
