public class HelperMethodsTests
{
    private readonly Fixture _fixture = new Fixture();

    [Fact]
    public void GetDecimalValue_ShouldReturnDecimalOrZero()
    {
        // Arrange
        var columnName = _fixture.Create<string>();
        var decimalValue = _fixture.Create<decimal>();

        dynamic reader = new ExpandoObject();
        var dict = (IDictionary<string, object>)reader;
        dict[columnName] = decimalValue;

        // Method info
        var method = typeof(CurrentServices).GetMethod(
            "GetDecimalValue",
            BindingFlags.NonPublic | BindingFlags.Static);
        Assert.NotNull(method);

        // Act
        var result = (decimal)method!.Invoke(null, new object[] { reader, columnName });
        Assert.Equal(decimalValue, result);

        // Test DBNull
        dict[columnName] = DBNull.Value;
        var resultDefault = (decimal)method!.Invoke(null, new object[] { reader, columnName });
        Assert.Equal(0m, resultDefault);
    }

    [Fact]
    public void GetPMCodes_ShouldReturnQuotedPmCodesOrEmpty()
    {
        // Arrange
        var request = _fixture.Build<GetAffiliatePlantDistributionRequest>()
                              .With(x => x.kpiCode, ((int)KpiDetailConstants.mtbf).ToString())
                              .With(x => x.subCategory, _fixture.Create<string>())
                              .Create();

        var pmCodeList = _fixture.CreateMany<AssetClassSubCategory>(3).ToList();

        // Add a matching asset class
        var code1 = _fixture.Create<string>();
        var code2 = _fixture.Create<string>();
        pmCodeList.Add(new AssetClassSubCategory
        {
            assetClass = request.subCategory,
            pmCode = $"{code1},{code2}"
        });

        var method = typeof(CurrentServices).GetMethod(
            "GetPMCodes",
            BindingFlags.NonPublic | BindingFlags.Static);
        Assert.NotNull(method);

        // Act
        var result = (string)method!.Invoke(null, new object[] { request, pmCodeList })!;

        // Assert
        Assert.NotNull(result);
        Assert.Contains($"'{code1}'", result);
        Assert.Contains($"'{code2}'", result);
        Assert.Contains(",", result); // comma-separated

        // Case: non-integer kpiCode
        request.kpiCode = _fixture.Create<string>();
        var resultNonInt = (string)method!.Invoke(null, new object[] { request, pmCodeList })!;
        Assert.Equal("''", resultNonInt);

        // Case: kpiCode not mtbf/mttr
        request.kpiCode = "999";
        var resultInvalidKpi = (string)method!.Invoke(null, new object[] { request, pmCodeList })!;
        Assert.Equal("''", resultInvalidKpi);
    }
}
