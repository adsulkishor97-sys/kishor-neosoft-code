#region PerformanceSummaryAffiliateAsync
[Theory]
[ClassData(typeof(PerformanceSummaryAffiliateRequestGenerator))]
public async Task PerformanceSummaryAffiliateAsync_ShouldReturnData_WhenRepositoryReturnsData(GetKpiPerformanceRequest request, GetCaseHierarchyRequest affiliateIdrequest, List<AffiliateList> affiliateList,
     List<KpiFormulaTarget> dbFormulas, List<GetCaseHierarchyResponse> getCaseHierarchyResponses)
{
    // Arrange    

    _currRepositoryMock.Setup(repo => repo.GetAffiliateLists()).ReturnsAsync(affiliateList);

    _configRepositoryMock.Setup(repo => repo.GetCaseHierarchyAsync(It.IsAny<GetCaseHierarchyRequest>())).ReturnsAsync(getCaseHierarchyResponses);


    _performanceSumRepositoryMock.Setup(repo => repo.GetKpiFormulaTarget(It.IsAny<KpiFormulaTargetRequest>())).Returns(Task.FromResult(dbFormulas));

    //Act
    var result = await _performanceSummServices.PerformanceSummaryAffiliateAsync(request, affiliateIdrequest.affiliateIdList!);
    // Assert
    Assert.NotNull(result);
    Assert.IsType<KpiPerformanceResponse>(result);

}

#endregion
