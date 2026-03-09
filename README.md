public async Task<GetHierarchyInfoResponse?> GetHierarchyInfoAsync(GetHierarchyInfoRequest request, string sqlAffiliateRequest)
{
    var emptyResponse = new GetHierarchyInfoResponse();
    string plantRequest = string.Empty;
    List<int> tokenplantIds=new List<int>();
    if (request.plantId != null)
    {
        plantRequest = Convert.ToString(request.plantId.Value);
    }
    if (sqlAffiliateRequest == null)
    {
        return new GetHierarchyInfoResponse();
    }
    var httpContext = _httpContextAccessor.HttpContext;
    var user = httpContext!.User;
    var isValidAffiliate = CheckAffiliateValidation(request.affiliateId?.ToString() ?? "");
    var isValidPlant = CheckPlantValidation(request.plantId?.ToString() ?? "");
    string tokenplantIdList = "";
    await ValidData(request, isValidAffiliate, isValidPlant, user, emptyResponse);

    var lstPlants = await _assetRepository.GetCaseHierarchyAsync(Convert.ToInt32(request.affiliateId));
    bool plantFlag = true;
    string formateaffiliateIds = "";
    if (user.IsInRole(PageConstants.ADMIN) || user.IsInRole(PageConstants.CORPORATE) || (user.IsInRole(PageConstants.AFFILIATE) && isValidAffiliate))
    {
        var affiliateID = request.affiliateId.ToString() ?? "";
        var affiliateIds = affiliateID!.Split(',').Select(id => id.Trim()).Where(id => !string.IsNullOrEmpty(id)).ToList();
        formateaffiliateIds = string.Join(",", affiliateIds);
        plantFlag = false;
    }
    else
    {
        tokenplantIdList = string.Join(',', lstPlants.Where(a => tokenplantIds.Contains(a.plantId!)).Select(a => a.plantId).ToList());
        formateaffiliateIds = string.Join(',', lstPlants.Where(a => tokenplantIds.Contains(a.plantId!)).Select(a => a.affiliateId).ToList());
        plantFlag = true;
    }



    if (formateaffiliateIds != null && formateaffiliateIds.Length > 0)
    {
        var getAffiliateCodeList = GetAffiliateCodeList(request.affiliateId);

        sqlAffiliateRequest = $"{string.Join(",", getAffiliateCodeList.Select(n => $"{n}"))}";
    }
    string? requestedPlant = CheckValidatePlant(request, plantRequest, plantFlag, tokenplantIdList);
    
List<GetHierarchyInfoSqlDBResponse> getHierarchyInfoSqlDBResponse = await _configRepository.GetHierarchyInfoFromSqlDBAsync(request.page!, requestedPlant != null ? requestedPlant : sqlAffiliateRequest);

var finalResponse = new GetHierarchyInfoResponse
{
    affiliates = getHierarchyInfoSqlDBResponse.FirstOrDefault()!.affiliateCount,
    assetClass = getHierarchyInfoSqlDBResponse.FirstOrDefault()!.assetClassCount,
    plants = getHierarchyInfoSqlDBResponse.FirstOrDefault()!.plantCount,
    product = getHierarchyInfoSqlDBResponse.FirstOrDefault()!.productCount,
    assets = getHierarchyInfoSqlDBResponse.FirstOrDefault()!.overAllAsset,
    criticalAssets = getHierarchyInfoSqlDBResponse.FirstOrDefault()!.criticalAsset,
};
    return finalResponse;
}

 [Fact]

 public async Task GetHierarchyInfoAsync_ShouldReturnMappedResponse_WhenPlantIdIsProvided()
 {
     // Arrange
     var fixture = new Fixture();
     var request = fixture.Create<GetHierarchyInfoRequest>();            
     string sqlAffiliateRequest = It.IsAny<string>();


     // Act
     var result = await _mockconfigServices.GetHierarchyInfoAsync(request, sqlAffiliateRequest);

     // Assert
     Assert.NotNull(result);
     Assert.IsType<GetHierarchyInfoResponse>(result);
 }
