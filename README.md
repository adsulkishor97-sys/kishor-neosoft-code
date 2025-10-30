[Fact]
    public void GetDeciamlSafe_ShouldReturnDecimal_WhenValueIsNotDBNull()
    {
        // Arrange
        var columnName = _fixture.Create<string>();
        var expectedValue = _fixture.Create<decimal>();

        // Mock DbDataReader
        var mockReader = new Mock<DbDataReader>();
        mockReader.Setup(r => r[columnName]).Returns(expectedValue);

        // Use reflection to access private static method
        var method = typeof(YourClassName)
            .GetMethod("GetDeciamlSafe", BindingFlags.NonPublic | BindingFlags.Static);
        Assert.NotNull(method);

        // Act
        var result = method.Invoke(null, new object[] { mockReader.Object, columnName });

        // Assert
        Assert.Equal(expectedValue, (decimal)result);
    }

    [Fact]
    public void GetDeciamlSafe_ShouldReturnZero_WhenValueIsDBNull()
    {
        // Arrange
        var columnName = _fixture.Create<string>();

        var mockReader = new Mock<DbDataReader>();
        mockReader.Setup(r => r[columnName]).Returns(DBNull.Value);

        var method = typeof(YourClassName)
            .GetMethod("GetDeciamlSafe", BindingFlags.NonPublic | BindingFlags.Static);
        Assert.NotNull(method);

        // Act
        var result = method.Invoke(null, new object[] { mockReader.Object, columnName });

        // Assert
        Assert.Equal(0m, (decimal)result);
    }
