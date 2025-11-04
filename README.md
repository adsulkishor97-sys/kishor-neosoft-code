[Fact]
public void GetParsedDate_ShouldReturnParsedDate_WhenInputIsValid()
{
    // Arrange
    var inputDate = _fixture.Create<DateTime>().ToString(CultureInfo.InvariantCulture);

    var method = typeof(CurrentServices)
        .GetMethod("GetParsedDate", BindingFlags.NonPublic | BindingFlags.Static);
    Assert.NotNull(method);

    // Act
    var result = method.Invoke(null, new object[] { inputDate });

    // Assert
    Assert.NotNull(result);
    Assert.IsType<DateTime>(result);
    Assert.Equal(DateTime.Parse(inputDate, CultureInfo.InvariantCulture), (DateTime)result);
}
