﻿# NuGet Credential Providers for Visual Studio

The NuGet Visual Studio Extension 3.6+ now supports Credential Providers, which enable NuGet to work seamlessly with authenticated feeds.
After you install a NuGet Credential Provider for Visual Studio, the NuGet Visual Studio extension will automatically acquire and refresh credentials for authenticated feeds as necessary.

Note: NuGet Credential Providers for Visual Studio only work in Visual Studio (not in dotnet restore or nuget.exe). To learn more about NuGet.exe Credential Providers, check out [this page](NuGet.exe-Credential-Providers.md).

NuGet Credential Providers for Visual Studio must be installed as a regular Visual Studio extension.

## Available NuGet Credential Providers for Visual Studio

There is a single credential provider that is built into the Visual Studio NuGet extension to support Visual Studio Team Services.

The NuGet Visual Studio Extension uses an internal `VsCredentialProviderImporter` which also scans for plug-in credential providers. These plug-in credential providers must be discoverable as a MEF Export of type `IVsCredentialProvider`.

## Creating a NuGet Credential Provider for Visual Studio

The NuGet Visual Studio Extension 3.6+ implements an internal CredentialService that is used to acquire credentials. The CredentialService has a list of built-in and plug-in Credential Providers. Each provider is tried sequentially until credentials are acquired.

During credential acquisition, the credential service will try credential providers in the following order, stopping as soon as credentials are acquired:

a) Credentials will be fetched from NuGet configuration files (using the built-in `SettingsCredentialProvider`).
b) If the package source is on Visual Studio Team Services, the `VisualStudioAccountProvider` will be used.
c) All 3rd party plug-in credential providers will be tried in the order of discovery outlined above.
d) If no credentials have been acquired yet, the user will be prompted for credentials using a standard basic authentication dialog.

To create a NuGet Credential Provider for Visual Studio, create a Visual Studio Extension that exposes a public MEF Export implementing the `IVsCredentialProvider` type, and adheres to the principles outlined below.
A sample implementation can be found in [our GitHub Samples](https://github.com/NuGet/Samples) repository.

### NuGet Credential Provider for Visual Studio Basics

Each NuGet Credential Provider for Visual Studio must:

a) Determine whether it can provide credentials for the targeted URI before initiating credential acquisition. If the provider cannot supply credentials for the targeted source, then it should return `null`.
b) If the provider does handle requests for the targeted URI, but cannot supply credentials, an exception should be thrown.

### NuGet Credential Provider for Visual Studio Input Parameters

A custom NuGet Credential Provider for Visual Studio must implement the `IVsCredentialProvider.GetCredentialsAsync` method accepting the following input parameters:

<table>
<th>Input parameter</th>
<th>Description</th>
    <tr>
        <td>Uri uri</td>
        <td>The package source Uri for which credentials are being requested.</td>
    </tr>
    <tr>
        <td>IWebProxy proxy</td>
        <td>Web proxy to use when communicating on the network. Null if there is no proxy authentication configured.</td>
    </tr>
    <tr>
        <td>bool isProxyRequest</td>
        <td>True if this request is to get proxy authentication credentials. If the implementation is not valid for acquiring proxy credentials, then null should be returned.</td>
    </tr>
    <tr>
        <td>bool isRetry</td>
        <td>True if credentials were previously requested for this Uri, but the supplied credentials did not allow authorized access.</td>
    </tr>
    <tr>
        <td>bool nonInteractive</td>
        <td>If true, then interactive prompts must not be allowed.</td>
    </tr>
    <tr>
        <td>CancellationToken cancellationToken</td>
        <td>This cancellation token should be checked to determine if the operation requesting credentials has been cancelled.</td>
    </tr>
</table>

### NuGet Credential Provider for Visual Studio Output Parameter

The result of the call to `IVsCredentialProvider.GetCredentialsAsync` is a credentials object implementing the `System.Net.ICredentials` interface.

More information about this type can be found [here](https://msdn.microsoft.com/en-us/library/system.net.icredentials(v=vs.110).aspx).