using System;
using System.Globalization;
using System.Reflection;
using AutoFixture;
using Xunit;

public class ReplaceQueryPrivateTests
{
    [Fact]
    public void ReplaceQuery_PrivateStatic_ShouldReplaceAllPlaceholders_WithDynamicValues()
    {
        // Arrange
        var fixture = new Fixture();

        var queryTemplate = "SELECT * FROM KPI WHERE affiliateId IN (${affiliateIdList}) " +
                            "AND plantId IN (${plantIdList}) " +
                            "AND sapId = ${sapId} " +
                            "AND templateId = ${templateId} " +
                            "AND start_date BETWEEN ${startDate} AND ${endDate} " +
                            "AND yyyymm BETWEEN ${startDateyyyymm} AND ${endDateyyyymm} " +
                            "AND yyyymmdd BETWEEN ${startDateyyyymmdd} AND ${endDateyyyymmdd};";

        var startDate = fixture.Create<DateTime>().AddDays(-30).ToString("yyyy-MM-dd", CultureInfo.InvariantCulture);
        var endDate = fixture.Create<DateTime>().ToString("yyyy-MM-dd", CultureInfo.InvariantCulture);
        var affiliateRequest1 = fixture.Create<string>();
        var plantIds = fixture.Create<string>();
        var sapIds = fixture.Create<string>();
        var templateId = fixture.Create<string>();

        // Get type of class under test (replace `YourServiceClass` with actual class name)
        var targetType = typeof(YourServiceClass);

        // Get private static method info
        var methodInfo = targetType.GetMethod("ReplaceQuery",
            BindingFlags.NonPublic | BindingFlags.Static);

        Assert.NotNull(methodInfo); // ensure method found

        // Act
        var result = methodInfo!.Invoke(null, new object[]
        {
            queryTemplate, startDate, endDate, affiliateRequest1, plantIds, sapIds, templateId
        }) as string;

        // Assert
        Assert.False(string.IsNullOrWhiteSpace(result));
        Assert.DoesNotContain("${affiliateIdList}", result);
        Assert.DoesNotContain("${startDate}", result);
        Assert.DoesNotContain("${endDate}", result);
        Assert.DoesNotContain("${plantIdList}", result);
        Assert.DoesNotContain("${sapId}", result);
        Assert.DoesNotContain("${templateId}", result);
        Assert.DoesNotContain("${startDateyyyymm}", result);
        Assert.DoesNotContain("${endDateyyyymm}", result);
        Assert.DoesNotContain("${startDateyyyymmdd}", result);
        Assert.DoesNotContain("${endDateyyyymmdd}", result);

        Assert.Contains(affiliateRequest1, result);
        Assert.Contains(sapIds, result);
        Assert.Contains(templateId, result);

        var expectedStartDate = DateTime.Parse(startDate).ToString("yyyy-MM-dd");
        var expectedEndDate = DateTime.Parse(endDate).ToString("yyyy-MM-dd");
        Assert.Contains(expectedStartDate, result);
        Assert.Contains(expectedEndDate, result);
    }
}
