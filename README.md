#region PerformanceSummaryAsync
[Theory]
[ClassData(typeof(PerformanceSummaryRequestGenerator))]
public async Task PerformanceSummaryAsync_ShouldReturnData_WhenRepositoryReturnsData(GetKpiPerformanceRequest request, GetCaseHierarchyRequest affiliateIdrequest, List<AffiliateList> affiliateList
    , List<KpiFormulaTarget> dbFormulas)
{
    // Arrange    

    _currRepositoryMock.Setup(repo => repo.GetAffiliateLists()).ReturnsAsync(affiliateList);

    _performanceSumRepositoryMock.Setup(repo => repo.GetKpiFormulaTarget(It.IsAny<KpiFormulaTargetRequest>())).Returns(Task.FromResult(dbFormulas));
    //Act
    var result = await _performanceSummServices.PerformanceSummaryAsync(request, affiliateIdrequest.affiliateIdList!);
    // Assert
    Assert.NotNull(result);
    Assert.IsType<KpiPerformanceResponse>(result);

}

#endregion
