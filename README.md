using System;
using System.Globalization;
using System.Reflection;
using AutoFixture;
using Xunit;

public class YourClassTests
{
    private readonly Fixture _fixture = new Fixture();

    [Fact]
    public void ParseAndFormatDates_ShouldReturnFormattedDates_WhenValidInput()
    {
        // Arrange
        var startDate = _fixture.Create<DateTime>().ToString("yyyy-MM-dd", CultureInfo.InvariantCulture);
        var endDate = _fixture.Create<DateTime>().AddDays(5).ToString("yyyy-MM-dd", CultureInfo.InvariantCulture);

        // Get method info using reflection
        var methodInfo = typeof(YourClassName) // Replace with your actual class name
            .GetMethod("ParseAndFormatDates", BindingFlags.NonPublic | BindingFlags.Static);

        Assert.NotNull(methodInfo);

        // Act
        var result = methodInfo.Invoke(null, new object[] { startDate, endDate });

        // Assert
        Assert.NotNull(result);

        // Deconstruct the tuple
        var (formattedStartDate, formattedEndDate, startDateTimeDate, endDateTimeDate)
            = ((string, string, string, string))result!;

        Assert.Equal(DateTime.Parse(startDate).ToString("yyyy-MM-dd"), formattedStartDate);
        Assert.Equal(DateTime.Parse(endDate).ToString("yyyy-MM-dd"), formattedEndDate);
        Assert.Equal(DateTime.Parse(startDate).ToString("yyyyMM"), startDateTimeDate);
        Assert.Equal(DateTime.Parse(endDate).ToString("yyyyMM"), endDateTimeDate);
    }
}
