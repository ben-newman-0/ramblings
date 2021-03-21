---
title: "Disable Metadata Source in Synology's Video Station"
tags: [Metadata, Synology, Video Station]
---

While using the Video Station application on my Synology NAS I found that the metadata for some TV shows would be mixed from different data sources (such as IMDB and The TVDB). On certain shows this caused the metadata to contain different languages. It also caused some episodes from the same show to be treated as different shows.

To solve this problem I wanted to make Video Station use only one source for metadata. Unfortunately there was no way to do this from the options in Video Station. In the end I found that I could edit the code for a specific data source to simply return no results.

The following steps show how I disabled a specific Video Station metadata source.
<h1>1. Enable SSH</h1>
To change Video Station's metadata retrieval code you need to have SSH access to your NAS. If SSH is already enabled on the NAS these steps can be skipped.
<ol>
 	<li>Log in to the DSM web manager of the Synology NAS.</li>
 	<li>Launch the <strong>Control Panel</strong> application.</li>
 	<li>Go to the <strong>Terminal &amp; SNMP</strong> section.</li>
 	<li>Tick the <strong>Enable SSH service</strong> option. Ensure the port is set to 22.
<a href="/blog/wp-content/uploads/2015/02/controlpanelssh.png"><img class="alignnone size-large wp-image-25" src="/blog/wp-content/uploads/2015/02/controlpanelssh.png?w=660" alt="ControlPanelSSH" width="660" height="209" /></a></li>
 	<li>Click <strong>Apply</strong>.</li>
</ol>
<h1>2. Log In to the NAS via SSH</h1>
To log in to the SSH server on the NAS an SSH client is required, such as <a title="PuTTY" href="http://www.chiark.greenend.org.uk/~sgtatham/putty/" target="_blank">PuTTY</a> for Windows and the SSH terminal command for OS X / Linux.
<ol>
 	<li>Log in to the SSH server on the NAS using the following credentials:
<table>
<tbody>
<tr>
<td><strong>Host</strong></td>
<td>[IP address of the NAS]</td>
</tr>
<tr>
<td><strong>User Name</strong></td>
<td>root</td>
</tr>
<tr>
<td><strong>Password</strong></td>
<td>[Password of the administrator account]</td>
</tr>
</tbody>
</table>
</li>
 	<li>Change directory to the install location of Video Station by entering the following command.
<pre>cd /volume1/@appstore/VideoStation/plugins/</pre>
<em>Note: Replace <strong>volume1</strong> with the volume where the Video Station package was installed to. This can be found out via the Package Center.</em></li>
</ol>
<h1>3. Edit The Metadata Provider</h1>
Finally edit the metadata provider search code to return no results.
<ol>
 	<li>Use the ls command to see available providers. The following example disables the syno_synovideodb source. Change to the directory of the provider to be disabled using the following command.
<pre>cd syno_synovideodb/</pre>
</li>
 	<li>Open the search code file using the vi text editor.
<pre>vi search.php</pre>
</li>
 	<li>Use the arrow keys to find the <strong>Process</strong> function. A snippet of the start of the function for syno_synovideodb is shown below.
<pre> }
}

function Process($input, $lang, $type, $limit, $search_properties, $allowguess)
{
 global $DATA_TEMPLATE;
 //Init</pre>
</li>
 	<li>Press the <strong>I</strong> key to enter insert mode (the letter I should appear in the bottom-left corner of the terminal).</li>
 	<li>Immediately after the opening curly brace after the Process function declaration enter the following line of code:
<pre>return array();</pre>
This line of code will make the metadata provider return 0 results as soon as it is queried for data. Below is a snippet of the top of the Process function after adding the line of code.
<pre> }
}

function Process($input, $lang, $type, $limit, $search_properties, $allowguess)
{
 return array();
 global $DATA_TEMPLATE;
 //Init</pre>
</li>
 	<li>Hit the ESC key to exit insert mode. Save the changes and quit by entering <strong>:wq</strong> and pressing ENTER.</li>
</ol>
At this point the metadata provider should always return 0 results when it is searched. Note that any changes made may be lost when the Video Station package updates. For security reasons you should log back in to the web interface of the NAS and disable SSH.
