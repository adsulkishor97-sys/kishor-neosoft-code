using System;
using System.Globalization;
using AutoFixture;
using Xunit;

public class ReplaceQueryTests
{
    [Fact]
    public void ReplaceQuery_ShouldReplaceAllPlaceholders_WithDynamicValues()
    {
        // Arrange
        var fixture = new Fixture();

        // Generate random but realistic test data
        var queryTemplate = "SELECT * FROM KpiTable WHERE affiliateId IN (${affiliateIdList}) " +
                            "AND plantId IN (${plantIdList}) " +
                            "AND sapId = ${sapId} " +
                            "AND templateId = ${templateId} " +
                            "AND period BETWEEN ${startDate} AND ${endDate} " +
                            "AND yyyymm BETWEEN ${startDateyyyymm} AND ${endDateyyyymm} " +
                            "AND yyyymmdd BETWEEN ${startDateyyyymmdd} AND ${endDateyyyymmdd};";

        var startDate = fixture.Create<DateTime>().AddDays(-10).ToString("yyyy-MM-dd", CultureInfo.InvariantCulture);
        var endDate = fixture.Create<DateTime>().ToString("yyyy-MM-dd", CultureInfo.InvariantCulture);
        var affiliateRequest1 = fixture.Create<string>();
        var plantIds = fixture.Create<string>();
        var sapIds = fixture.Create<string>();
        var templateId = fixture.Create<string>();

        // Act
        var result = ReplaceQuery(queryTemplate, startDate, endDate, affiliateRequest1, plantIds, sapIds, templateId);

        // Assert
        Assert.False(string.IsNullOrWhiteSpace(result)); // should not be null or empty
        Assert.DoesNotContain("${affiliateIdList}", result);
        Assert.DoesNotContain("${startDate}", result);
        Assert.DoesNotContain("${endDate}", result);
        Assert.DoesNotContain("${plantIdList}", result);
        Assert.DoesNotContain("${sapId}", result);
        Assert.DoesNotContain("${templateId}", result);
        Assert.DoesNotContain("${startDateyyyymmdd}", result);
        Assert.DoesNotContain("${endDateyyyymmdd}", result);

        // Optional: ensure formatted replacements are correct
        Assert.Contains(affiliateRequest1, result);
        Assert.Contains(sapIds, result);
        Assert.Contains(templateId, result);

        var formattedStart = DateTime.Parse(startDate).ToString("yyyy-MM-dd");
        var formattedEnd = DateTime.Parse(endDate).ToString("yyyy-MM-dd");

        Assert.Contains(formattedStart, result);
        Assert.Contains(formattedEnd, result);
    }

    // The actual method under test
    private static string ReplaceQuery(string? query, string startDate, string endDate, string affiliateRequest1, string plantIds, string sapIds, string templateId)
    {
        var startDateTime = DateTime.Parse(startDate, CultureInfo.InvariantCulture);
        var endDateTime = DateTime.Parse(endDate, CultureInfo.InvariantCulture);

        var formatedStartdate = "'" + startDateTime.ToString("yyyyMM") + "'";
        var formatedEnddate = "'" + endDateTime.ToString("yyyyMM") + "'";

        var formatedStartdateyyyymmdd = "'" + startDateTime.ToString("yyyyMMdd") + "'";
        var formatedEnddateyyyymmdd = "'" + endDateTime.ToString("yyyyMMdd") + "'";

        var finalQuery = query!
            .Replace("${affiliateIdList}", affiliateRequest1)
            .Replace("${startDate}", "'" + startDateTime.ToString("yyyy-MM-dd") + "'")
            .Replace("${endDate}", "'" + endDateTime.ToString("yyyy-MM-dd") + "'")
            .Replace("${startDateyyyymmdd}", formatedStartdateyyyymmdd)
            .Replace("${endDateyyyymmdd}", formatedEnddateyyyymmdd)
            .Replace("${plantIdList}", plantIds)
            .Replace("${startDateyyyymm}", formatedStartdate)
            .Replace("${endDateyyyymm}", formatedEnddate)
            .Replace("${sapId}", sapIds)
            .Replace("${templateId}", templateId);

        return finalQuery;
    }
}
