[Fact]
public void GetQuotedPmCodesIfNeeded_ShouldReturnQuotedPmCodes_WhenKpiCodeMatches()
{
    // Arrange
    var fixture = new Fixture();

    // Create a fake request
    var request = fixture.Build<GetAssetDistributionRequest>()
                         .With(x => x.kpiCode, ((int)KpiDetailConstants.mtbf).ToString())
                         .With(x => x.subCategory, fixture.Create<string>())
                         .Create();

    // Create fake SQL data
    var pmCodeList = fixture.CreateMany<AssetClassSubCategory>(3).ToList();

    // Ensure at least one item matches the subCategory
    pmCodeList.Add(new AssetClassSubCategory
    {
        assetClass = request.subCategory,
        pmCode = $"{fixture.Create<string>()},{fixture.Create<string>()}"
    });

    // Get the method info for private static method
    var method = typeof(CurrentServices).GetMethod(
        "GetQuotedPmCodesIfNeeded",
        BindingFlags.NonPublic | BindingFlags.Static);

    Assert.NotNull(method);

    // Act
    var result = (string)method!.Invoke(null, new object[] { request, pmCodeList })!;

    // Assert
    Assert.NotNull(result);
    Assert.Contains("'", result); // it should have single quotes around codes
}

[Fact]
public void GetQuotedPmCodesIfNeeded_ShouldReturnEmptyQuotes_WhenKpiCodeNotMatching()
{
    // Arrange
    var fixture = new Fixture();

    var request = fixture.Build<GetAssetDistributionRequest>()
                         .With(x => x.kpiCode, "999") // non-mtbf/mttr
                         .With(x => x.subCategory, fixture.Create<string>())
                         .Create();

    var pmCodeList = fixture.CreateMany<AssetClassSubCategory>(2).ToList();

    var method = typeof(CurrentServices).GetMethod(
        "GetQuotedPmCodesIfNeeded",
        BindingFlags.NonPublic | BindingFlags.Static);

    Assert.NotNull(method);

    // Act
    var result = (string)method!.Invoke(null, new object[] { request, pmCodeList })!;

    // Assert
    Assert.Equal("''", result);
}

[Fact]
public void GetQuotedPmCodesIfNeeded_ShouldReturnEmptyQuotes_WhenKpiCodeNotInt()
{
    // Arrange
    var fixture = new Fixture();

    var request = fixture.Build<GetAssetDistributionRequest>()
                         .With(x => x.kpiCode, fixture.Create<string>()) // non-integer
                         .With(x => x.subCategory, fixture.Create<string>())
                         .Create();

    var pmCodeList = fixture.CreateMany<AssetClassSubCategory>(2).ToList();

    var method = typeof(CurrentServices).GetMethod(
        "GetQuotedPmCodesIfNeeded",
        BindingFlags.NonPublic | BindingFlags.Static);

    Assert.NotNull(method);

    // Act
    var result = (string)method!.Invoke(null, new object[] { request, pmCodeList })!;

    // Assert
    Assert.Equal("''", result);
}
