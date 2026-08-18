# Creating an Entra application for website sign-in

In order to allow users to sign in to your demonstration website, you'll need to create an application in Entra ID. You can do this by running on the following command on az CLI:

```
az ad app create \
    --display-name "ChaosTesting-DemoWebsite" \
    --sign-in-audience "AzureADMyOrg" \
    --required-resource-accesses '[{"resourceAppId":"00000003-0000-0000-c000-000000000000","resourceAccess":[{"id":"e1fe6dd8-ba31-4d61-89e7-88639da4683d","type":"Scope"}]}]'
```

Copy the `appId` returned by the command, then add the deployed website origin as a
Single-page application redirect URI:

### Redirect URL
You'll need to replace the value for the Redirect URL on this application after you successfully deploy your application. This is the location to which Entra ID redirects after performing the authentication step. The correct value for this is shown in Github actions after running the deployment: 

![Deployment complete](../assets/deployment-complete.png)

You can update your redirect URL through the portal, or by using Microsoft Graph through the Azure CLI. The first command resolves the application's object ID from its client ID:
```powershell
$appObjectId = az ad app show `
    --id <application-id> `
    --query id `
    --output tsv

$body = '{"spa":{"redirectUris":["https://my-new-url.azurefd.net"]}}'
$payloadPath = New-TemporaryFile
Set-Content -Path $payloadPath -Value $body -Encoding utf8

az rest `
    --method PATCH `
    --url "https://graph.microsoft.com/v1.0/applications/$appObjectId" `
    --headers "Content-Type=application/json" `
    --body "@$payloadPath"

Remove-Item $payloadPath -Force
```

### Notes
- The website uses MSAL Browser, which signs users in with the authorization-code flow with PKCE. Add the deployed website origin under the application's **Single-page application** redirect URIs, not under **Web**.
- The values for 'required-resource-accesses' represent the User.Read scope, allowing the application to retrieve the basic user profile. The user will have to consent to the application receiving this scope upon the first login. 