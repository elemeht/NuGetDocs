# NuGet.exe Credential Providers

NuGet.exe 3.3+ now supports Credential Providers, which enable NuGet to work seamlessly with authenticated feeds. 
After you install a NuGet.exe Credential Provider, NuGet.exe will automatically acquire and refresh credentials for authenticated feeds as necessary.

Note: NuGet.exe Credential providers only work in NuGet.exe (not in dotnet restore or Visual Studio). To learn more about NuGet Credential Providers for Visual Studio, check out [this page](Visual-Studio-Credential-Providers.md).

NuGet.exe Credential Providers can be used in 3 ways:

* [Globally](#installing-a-credential-provider-globally)

* [Environment Variable](#using-a-credential-provider-from-an-environment-variable)

* [Alongside NuGet.exe](#using-a-credential-provider-alongside-nugetexe)

## Installing a NuGet.exe Credential Provider Globally

To make a Credential Provider available to all instances of NuGet.exe run under the current user's profile, 
add it to `%LocalAppData%\NuGet\CredentialProviders`

You may need to create the `CredentialProviders` folder.

NuGet.exe Credential Providers can be installed at the root of the `CredentialProviders` folder or within a subfolder. If a NuGet.exe Credential Provider has multiple files/assemblies, using subfolders can help keep the Providers organized.

## Using a NuGet.exe Credential Provider from an Environment Variable

NuGet.exe Credential Providers can also be stored anywhere and then made accessible to NuGet.exe via an environment variable. To use a NuGet.exe Credential Provider this way, set the `%NUGET_CREDENTIALPROVIDERS_PATH%` to the location of your Provider. The variable can be a semicolon-separated list, e.g. `path1;path2`, if you have have multiple NuGet.exe Credential Providers in different locations.

## Using a NuGet.exe Credential Provider Alongside NuGet.exe

NuGet.exe Credential Providers can also be placed in the same folder as NuGet.exe.

## Using Multiple NuGet.exe Credential Providers

The NuGet.exe client will search the above locations, in order, for NuGet.exe Credential Providers. Specifically, it will search for any file that matches the pattern `CredentialProvider*.exe` and load each provider in the order it's found. If two NuGet.exe Credential Providers are found in the same directory, they will be loaded in alphabetical order.

## Available NuGet.exe Credential Providers

Some of the available NuGet.exe Credential Providers are: [Visual Studio Team Services Credential Provider](https://www.visualstudio.com/en-us/docs/package/get-started/nuget/auth#vsts-credential-provider)

## Creating a NuGet.exe Credential Provider

The NuGet.exe 3.3 client implements an internal CredentialService that is used to acquire credentials. The CredentialService has a list of built-in and plug-in NuGet.exe Credential Providers. Each provider is tried sequentially until credentials are acquired.

Built-in providers, such as the existing providers to fetch credentials from nuget.config and to
prompt the user on the command line, execute in-process. 

During credential acquisition, the credential service will try plug-in credential providers in the following order, stopping as soon as credentials are acquired:

a) Credentials will be fetched from NuGet configuration files.

b) All 3rd party credential providers will be tried in the order of discovery outlined above.

c) The user will be prompted for credentials on the command line.

To create a NuGet.exe Credential Provider, create a command-line executable that takes the inputs specified below, acquires credentials as appropriate, then returns the appropriate exit status code and standard output.

### NuGet.exe Credential Provider Executable Basics

NuGet.exe Credential Providers must follow the naming convention `CredentialProvider*.exe`., and each Credential Provider must:

a) Determine whether it can provide credentials for the targeted URI before initiating credential acquisition. If the plugin cannot supply credentials for the targeted source, then it should return
   status code 1 and supply no credentials.

b) Not modify nuget.config (i.e. by setting credentials there).

c) Handle HTTP Proxy configuration on their own. The CredentialService will not pass HTTP Proxy configuration information to the plugin.

d) Encode stdout responses using UTF-8 encoding.

### NuGet.exe Credential Provider Input Parameters

<table>
<th>Input parameter</th>
<th>Description</th>
    <tr>
        <td>Uri{value}</td>
        <td>The package source Uri for which credentials will be filled.</td>
    </tr>
    <tr>
        <td>NonInteractive</td>
        <td>If present, providers will not issue interactive prompts.</td>
    </tr>
    <tr>
        <td>IsRetry</td>
        <td>If present, this is a retry and the credentials were rejected on a previous attempt. Providers typically use the `IsRetry` flag to ensure that they bypass any existing cache and prompt for new credentials if possible.</td>
    </tr>
</table>

### NuGet.exe Credential Provider Exit Status Codes

<table>
<th>Exit code</th>
<th>Result</th>
<th>Description</th>
    <tr>
        <td>0</td>
        <td>Success</td>
        <td>Credentials were successfully acquired and have been written to stdout.</td>
    </tr>
    <tr>
        <td>1</td>
        <td>ProviderNotApplicable</td>
        <td>The current provider does not provide credentials for the given uri.</td>
    </tr>
    <tr>
        <td>2</td>
        <td>Failure</td>
        <td>The NuGet.exe Credential Provider is the correct provider for the given Uri, but cannot provide credentials. In this case, the CredentialService will throw an exception and the current request should fail and not be retried. A typical example would be a user cancelling an interactive login.</td>
    </tr>
</table>

### NuGet.exe Credential Provider Standard Output

<table>
<th>Property</th>
<th>Notes</th>
    <tr>
        <td>Username</td>
        <td>Username for authenticated requests.
        </td>
    </tr>
    <tr>
        <td>Password</td>
        <td>Password for authenticated requests.</td>
    </tr>
    <tr>
        <td>Message</td>
        <td>Optional details about the response. Will only be used by NuGet to show additional error details in failure cases.</td>
    </tr>
</table>

### Example stdout

    { "Username" : "freddy@example.com",
      "Password" : "bwm3bcx6txhprzmxhl2x63mdsul6grctazoomtdb6kfbof7m3a3z",
      "Message"  : "" }

