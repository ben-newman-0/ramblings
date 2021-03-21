---
title: "Disable Metadata Source in Synology's Video Station"
tags: [Metadata, Synology, Video Station]
---

## Disable Metadata Source in Synology's Video Station

While using the Video Station application on my Synology NAS I found that the metadata for some TV shows would be mixed from different data sources (such as IMDB and The TVDB). On certain shows this caused the metadata to contain different languages. It also caused some episodes from the same show to be treated as different shows.

To solve this problem I wanted to make Video Station use only one source for metadata. Unfortunately there was no way to do this from the options in Video Station. In the end I found that I could edit the code for a specific data source to simply return no results.

The following steps show how I disabled a specific Video Station metadata source.

### 1. Enable SSH

To change Video Station's metadata retrieval code you need to have SSH access to your NAS. If SSH is already enabled on the NAS these steps can be skipped.

1. Log in to the DSM web manager of the Synology NAS.
1. Launch the **Control Panel** application.
1. Go to the **Terminal & SNMP** section.
1. Tick the **Enable SSH service** option. Ensure the port is set to 22.
    ![Screenshot of SSH settings in Control Panel](/assets/posts/2015-03-10-disable-metadata-source-in-synologys-video-station/controlpanelssh.png)
1. Click **Apply**.

### 2. Log In to the NAS via SSH

To log in to the SSH server on the NAS an SSH client is required, such as [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/) for Windows and the SSH terminal command for OS X / Linux.

1. Log in to the SSH server on the NAS using the following credentials:

    | Property  | Value                                   |
    | --------- | --------------------------------------- |
    | Host      | *IP address of the NAS*                 |
    | User Name | root                                    |
    | Password  | *Password of the administrator account* |

1. Change directory to the install location of Video Station by entering the following command.

    ```bash
    cd /volume1/@appstore/VideoStation/plugins/
    ```

    *Note: Replace **volume1** with the volume where the Video Station package was installed to. This can be found out via the Package Center.*

### 3. Edit The Metadata Provider

Finally edit the metadata provider search code to return no results.

1. Use the ls command to see available providers. The following example disables the syno_synovideodb source. Change to the directory of the provider to be disabled using the following command.

    ```bash
    cd syno_synovideodb/
    ```

1. Open the search code file using the vi text editor.

    ```bash
    vi search.php
    ```

1. Use the arrow keys to find the **Process** function. A snippet of the start of the function for syno_synovideodb is shown below.

    ```php
        }
    }

    function Process($input, $lang, $type, $limit, $search_properties, $allowguess)
    {
        global $DATA_TEMPLATE;
        //Init
    ```

1. Press the **I** key to enter insert mode (the letter I should appear in the bottom-left corner of the terminal).

1. Immediately after the opening curly brace after the Process function declaration enter the following line of code:

    ```php
    return array();
    ```

    This line of code will make the metadata provider return 0 results as soon as it is queried for data. Below is a snippet of the top of the Process function after adding the line of code.

    ```php
        }
    }

    function Process($input, $lang, $type, $limit, $search_properties, $allowguess)
    {
        return array();
        global $DATA_TEMPLATE;
        //Init
    ```

1. Hit the ESC key to exit insert mode. Save the changes and quit by entering **:wq** and pressing ENTER.

At this point the metadata provider should always return 0 results when it is searched. Note that any changes made may be lost when the Video Station package updates. For security reasons you should log back in to the web interface of the NAS and disable SSH.
