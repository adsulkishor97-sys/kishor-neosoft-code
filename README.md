 public CaseHierarchyTokenAccessDetails GetCaseHierarchyTokenAccessDetails()
 {
     var httpContext = _httpContextAccessor.HttpContext;
     CryptographyHelper crypt = new(_jwtSettings);
     var authHeader = httpContext?.Request.Headers.Authorization.FirstOrDefault();
     var token = authHeader?["Bearer ".Length..];

     var handler = new JwtSecurityTokenHandler();
     var jwtToken = handler.ReadJwtToken(token);

     var user = httpContext!.User;

     if (user.IsInRole(Constants.admin) || user.IsInRole(Constants.corporate))
     {
         return new CaseHierarchyTokenAccessDetails
         {
             accessRole = "admin",
             tokenAffiliateIds=null,
             tokenPlantIds=null
         };
     }

     // get affiliates from Token
     List<string> affiliateData = new();
     var affiliateIDList = jwtToken.Claims.Where(x => x.Type == "affiliateID").Select(x => x.Value).ToList()!;
     affiliateIDList.ForEach(x =>
     {
         if (!string.IsNullOrEmpty(x))
         {
             affiliateData.Add(crypt.AesGcmDecrypt(x));
         }
     });

     // get plants from Token
     List<string> plantData = new();
     var plantIDList = jwtToken.Claims.Where(x => x.Type == "plantID").Select(x => x.Value).ToList()!;
     plantIDList.ForEach(x =>
     {
         if (!string.IsNullOrEmpty(x))
         {
             plantData.Add(crypt.AesGcmDecrypt(x));
         }
     });

     return new CaseHierarchyTokenAccessDetails
     {
         accessRole = "non-admin",
         tokenAffiliateIds = affiliateData,
         tokenPlantIds = plantData
     };
 }
