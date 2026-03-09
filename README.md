 private Task<GetHierarchyInfoResponse> ValidData(GetHierarchyInfoRequest request, bool isValidAffiliate, bool isValidPlant, ClaimsPrincipal user, GetHierarchyInfoResponse emptyResponse)
 {
     // user try to access affiliate page(means they are trying to access asset page/affiliate page)
     if (request.page == PageConstants.AFFILIATE)
     {
         var tokenplantIdList = GetPlantCodeList();
         var tokenplantIds = tokenplantIdList
                                            .Split(',', StringSplitOptions.RemoveEmptyEntries)
                                            .Select(id => int.TryParse(id, out var parsedId) ? parsedId : (int?)null)
                                            .Where(id => id.HasValue)
                                            .Select(id => id!.Value)
                                            .ToList();

         if(tokenplantIds.Count == 0)
         {
             return Task.FromResult(emptyResponse);
         }
     }
     // user has plant role and try to access global page
     else if ((user.IsInRole(PageConstants.PLANT) && request.page == PageConstants.GLOBAL) || (user.IsInRole(PageConstants.AFFILIATE) && !isValidAffiliate && !isValidPlant) || (user.IsInRole(PageConstants.PLANT) && !isValidPlant))
     {
         return Task.FromResult(emptyResponse);  // invalid access return 403
     }

     return Task.FromResult(emptyResponse);  

 }
