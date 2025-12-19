# Code to Generate GitHub Installation Access Token
```
#r "Newtonsoft.Json"
#r "Microsoft.Azure.Workflows.Scripting"
#r "System.IdentityModel.Tokens.Jwt"
#r "Microsoft.IdentityModel.Tokens"

using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Primitives;
using Microsoft.Extensions.Logging;
using Microsoft.Azure.Workflows.Scripting;
using Newtonsoft.Json.Linq;
using Newtonsoft.Json.Linq;
using Microsoft.IdentityModel.Tokens;
using System.IdentityModel.Tokens.Jwt;
using System.Security.Cryptography;
using System.Text;
using System.Net.Http.Headers;
using System.Text.Json;

/// <summary>
/// Executes the inline csharp code.
/// </summary>
/// <param name="context">The workflow context.</param>
/// <remarks> This is the entry-point to your code. The function signature should remain unchanged.</remarks>
public static async Task<Results> Run(WorkflowContext context, ILogger log)
{
  string AppId = "Your-App-Id";                    // e.g. 1851818
  long InstallationId = Your-Installation-Id; // e.g. 83026199
  string RepoOwner = "REPO-OWNER";
  string RepoName = ""REPO-NAME"";
  string WorkflowFile = "main.yml";
  string Branch = "main";
  
  JToken triggerOutputs = (await context.GetTriggerResults().ConfigureAwait(false)).Outputs;

  ////the following dereferences the 'name' property from trigger payload.
  var name = triggerOutputs?["body"]?["GetSecret"]?.ToString();

  var actionOutputs = (await context.GetActionResults("GetSecret").ConfigureAwait(false)).Outputs;
  var privateKeyPem  = actionOutputs["body"]["value"];
string signedToken = null;
using var rsa = RSA.Create();
{
    string actualKey = Encoding.UTF8.GetString(Convert.FromBase64String(privateKeyPem.ToString()));
    rsa.ImportFromPem(actualKey.ToCharArray());

    var credentials = new SigningCredentials(new RsaSecurityKey(rsa),
                                            SecurityAlgorithms.RsaSha256);
    var now = DateTimeOffset.UtcNow;
    long expiration = now.AddMinutes(10).ToUnixTimeMilliseconds();
    var jwtSecurityTokenHandler = new JwtSecurityTokenHandler();

    var rawToken = jwtSecurityTokenHandler.CreateToken(new SecurityTokenDescriptor
    {
      Issuer = AppId.ToString(),
      Expires = now.UtcDateTime.AddMinutes(10),
      IssuedAt = now.UtcDateTime,
     SigningCredentials = credentials
    });
    signedToken = jwtSecurityTokenHandler.WriteToken(rawToken);
}

string installToken = "";

using var http = new HttpClient();
{
    http.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", signedToken);
    http.DefaultRequestHeaders.UserAgent.Add(new ProductInfoHeaderValue("GHApp", "1.0"));

    var url = $"https://api.github.com/app/installations/{InstallationId}/access_tokens";
    var response = await http.PostAsync(url, null);
    response.EnsureSuccessStatusCode();

    using var doc = await JsonDocument.ParseAsync(await response.Content.ReadAsStreamAsync());
    installToken = doc.RootElement.GetProperty("token").GetString();
}
  return new Results
  {
    Message = $"{installToken}" 
  };
}

public class Results
{
  public string Message {get; set;}
}
```