 [Fact]
    public void GetParsedDate_ShouldReturnParsedDate_WhenInputIsValid()
    {
        // Arrange
        var inputDate = _fixture.Create<DateTime>().ToString(CultureInfo.InvariantCulture);

        var method = typeof(YourClassName)
            .GetMethod("GetParsedDate", BindingFlags.NonPublic | BindingFlags.Static);
        Assert.NotNull(method);

        // Act
        var result = method.Invoke(null, new object[] { inputDate });

        // Assert
        Assert.NotNull(result);
        Assert.IsType<DateTime>(result);
        Assert.Equal(DateTime.Parse(inputDate, CultureInfo.InvariantCulture), (DateTime)result);
    }

    [Fact]
    public void GetParsedDate_ShouldReturnNow_WhenInputIsNullOrWhitespace()
    {
        // Arrange
        string? input = " "; // whitespace case
        var method = typeof(YourClassName)
            .GetMethod("GetParsedDate", BindingFlags.NonPublic | BindingFlags.Static);
        Assert.NotNull(method);

        // Act
        var beforeCall = DateTime.Now;
        var result = (DateTime)method.Invoke(null, new object[] { input });
        var afterCall = DateTime.Now;

        // Assert — should be close to current time
        Assert.True(result >= beforeCall && result <= afterCall.AddSeconds(1));
    }
