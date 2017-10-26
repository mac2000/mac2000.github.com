---
layout: post
title: Generate custom JWT token for Firebase from dotnet core
tags: [dotnet, firebase, jwt]
---

In my case I were trying to marry our office Active Directory with [Firebase Custom Authentication](https://firebase.google.com/docs/auth/admin/create-custom-tokens)

That is preatty easy task if you are using anything except C# :)

In dotnet core at moment even talking to ldap is made via 3rd party library

After googling have found nice article: [Create JWT with a Private RSA Key](http://www.donaldsbaconbytes.com/2016/08/create-jwt-with-a-private-rsa-key/)


And after few attempts at least got working sample:


```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Security.Cryptography;
using Jose;
using Org.BouncyCastle.Crypto.Parameters;
using Org.BouncyCastle.OpenSsl;

namespace WebApplication1.Services
{
	/// <summary>
	/// Firebase Custom Token Generator
	///
	/// Authenticate with Firebase in JavaScript Using a Custom Authentication System
	/// Docs: https://firebase.google.com/docs/auth/web/custom-auth
	/// Client (firebase service account) email and private key can be retrieved here:
	/// https://console.firebase.google.com/project/_/settings/serviceaccounts/adminsdk
	///
	/// Required packages: BouncyCastle.NetCore, jose-jwt
	///
	/// Usage example:
	/// var firebase = new FirabaseCustomToken(clientEmail, privateKey);
	/// var token = firebase.CreateToken("alexandrm@rabota.ua");
	/// </summary>
	public class FirabaseCustomToken
	{
		private readonly string _clientEmail;
		private readonly string _privateKey;

		public FirabaseCustomToken(string clientEmail, string privateKey)
		{
			_privateKey = privateKey;
			_clientEmail = clientEmail;
		}

		public string CreateToken(string uid, Dictionary<string, object> claims = null)
		{
			var now = DateTimeOffset.Now.ToUnixTimeSeconds();

			var payload = new Dictionary<string, object>
			{
				{ "alg", "RS256" },
				{ "iss", _clientEmail },
				{ "sub", _clientEmail },
				{ "aud", "https://identitytoolkit.googleapis.com/google.identity.identitytoolkit.v1.IdentityToolkit" },
				{ "iat", now },
				{ "exp", now + 3600 },
				{ "uid", uid },
				{ "claims", claims ?? new Dictionary<string, object>() }
			};

			return SignToken(payload);
		}

		private string SignToken(Dictionary<string, object> payload)
		{
			string jwt;
			RsaPrivateCrtKeyParameters key;
			using (var stringReader = new StringReader(_privateKey))
			{
				var pemReader = new PemReader(stringReader);
				key = (RsaPrivateCrtKeyParameters)pemReader.ReadObject();
			}
			using (var rsa = new RSACryptoServiceProvider())
			{
				rsa.ImportParameters(ToRsaParameters(key));
				jwt = JWT.Encode(payload, rsa, JwsAlgorithm.RS256);
			}
			return jwt;
		}

		/// <summary>
		/// https://github.com/neoeinstein/bouncycastle/blob/master/crypto/src/security/DotNetUtilities.cs
		/// </summary>
		/// <param name="privKey">string</param>
		/// <returns></returns>
		private static RSAParameters ToRsaParameters(RsaPrivateCrtKeyParameters privKey) => new RSAParameters
		{
			Modulus = privKey.Modulus.ToByteArrayUnsigned(),
			Exponent = privKey.PublicExponent.ToByteArrayUnsigned(),
			D = privKey.Exponent.ToByteArrayUnsigned(),
			P = privKey.P.ToByteArrayUnsigned(),
			Q = privKey.Q.ToByteArrayUnsigned(),
			DP = privKey.DP.ToByteArrayUnsigned(),
			DQ = privKey.DQ.ToByteArrayUnsigned(),
			InverseQ = privKey.QInv.ToByteArrayUnsigned()
		};
	}
}
```


Further improvements may be accepting claims instead of dictionary, but unfortunatelly it seam that firebase itself does not respect claims from cutom tokens, so it is not an issue

As for Active Directory in .Net Core working sample is:


```csharp
using ActiveDirectoryJsonWebToken.Models;
using Microsoft.Extensions.Options;
using Novell.Directory.Ldap;
using System.Collections.Generic;
using System.Linq;
using System.Security.Claims;
using System.Security.Principal;

namespace ActiveDirectoryJsonWebToken.Services
{
	public class IdentityService
	{
		private readonly LdapConfig _config;
		public IdentityService(IOptions<LdapConfig> config)
		{
			_config = config.Value;
		}

		public ClaimsIdentity GetIdentity(LoginModel login)
		{
			var cn = new LdapConnection();
			try
			{
				cn.Connect(_config.Hostname, LdapConnection.DEFAULT_PORT);
				cn.Bind($"RABOTA\\{login.Username.Split('@').FirstOrDefault()}", login.Password);

				var claims = new List<Claim>();
				var results = cn.Search(_config.BaseDn, LdapConnection.SCOPE_ONE, $"(mail={login.Username})", null, false);
				var entry = results.next();

				claims.Add(new Claim(ClaimTypes.Email, login.Username));
				claims.Add(new Claim(ClaimTypes.GivenName, entry.getAttribute("name").StringValue));

				var groups = entry.getAttribute("MemberOf").StringValues;
				while (groups.MoveNext())
				{
					var group = groups.Current.ToString().Split(',').FirstOrDefault()?.Replace("CN=", "");
					claims.Add(new Claim(ClaimTypes.Role, group));
				}
				cn.Disconnect();

				return new ClaimsIdentity(new GenericIdentity(login.Username.ToLower()), claims);
			}
			catch (LdapException e)
			{
				if (e.ResultCode == LdapException.INVALID_CREDENTIALS)
				{
					return null;
				}

				throw e;
			}
		}
	}
}
```


It does require `Novell.Directory.Ldap.NETStandard` package to be installed to work

All is left is just to marry all this, but after playing around I have decided to run stuff in node windows container so having best from both worlds, leaving this just for future possible reuse
