[Fact]
public void ParseAndFormatDates_ShouldReturnFormattedDates_WhenValidInput()
{
    // Arrange
    var startDate = _fixture.Create<DateTime>().ToString(CultureInfo.InvariantCulture);
    var endDate = _fixture.Create<DateTime>().AddDays(5).ToString(CultureInfo.InvariantCulture);

    var method = typeof(CurrentServices)
        .GetMethod("ParseAndFormatDates", BindingFlags.NonPublic | BindingFlags.Static);
    Assert.NotNull(method);

    // Act
    var result = method.Invoke(null, new object[] { startDate, endDate });

    // Assert
    Assert.NotNull(result);
    Assert.IsType<ValueTuple<string, string, string, string>>(result);

    var (formattedStartDate, formattedEndDate, startDateTimeDate, endDateTimeDate)
        = ((string, string, string, string))result!;

    var expectedStart = DateTime.Parse(startDate, CultureInfo.InvariantCulture);
    var expectedEnd = DateTime.Parse(endDate, CultureInfo.InvariantCulture);

    Assert.Equal(expectedStart.ToString("yyyy-MM-dd"), formattedStartDate);
    Assert.Equal(expectedEnd.ToString("yyyy-MM-dd"), formattedEndDate);
    Assert.Equal(expectedStart.ToString("yyyyMM"), startDateTimeDate);
    Assert.Equal(expectedEnd.ToString("yyyyMM"), endDateTimeDate);
}
