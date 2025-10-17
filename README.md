public class HelperMethodsTests
{
    private readonly Fixture _fixture = new Fixture();

    [Fact]
    public void GetStringValue_ShouldReturnValueOrDefault()
    {
        // Arrange
        var columnName = _fixture.Create<string>();
        var expectedValue = _fixture.Create<string>();

        var readerMock = new Mock<DbDataReader>();
        readerMock.Setup(r => r[columnName]).Returns(expectedValue);

        var method = typeof(CurrentServices).GetMethod(
            "GetStringValue",
            BindingFlags.NonPublic | BindingFlags.Static);
        Assert.NotNull(method);

        // Act
        var result = (string?)method!.Invoke(null, new object[] { readerMock.Object, columnName });

        // Assert
        Assert.Equal(expectedValue, result);

        // Test DBNull
        readerMock.Setup(r => r[columnName]).Returns(DBNull.Value);
        var resultDefault = (string?)method!.Invoke(null, new object[] { readerMock.Object, columnName });
        Assert.Equal("0", resultDefault);
    }

    [Fact]
    public void GetIntegerValue_ShouldHandleValidAndSpecialValues()
    {
        // Arrange
        var columnName = _fixture.Create<string>();
        var validValue = _fixture.Create<long>();

        var readerMock = new Mock<DbDataReader>();
        readerMock.Setup(r => r[columnName]).Returns(validValue);

        var method = typeof(CurrentServices).GetMethod(
            "GetIntegerValue",
            BindingFlags.NonPublic | BindingFlags.Static);
        Assert.NotNull(method);

        // Act & Assert
        var result = (long?)method!.Invoke(null, new object[] { readerMock.Object, columnName });
        Assert.Equal(validValue, result);

        // DBNull case
        readerMock.Setup(r => r[columnName]).Returns(DBNull.Value);
        var resultDbNull = (long?)method!.Invoke(null, new object[] { readerMock.Object, columnName });
        Assert.Equal(0, resultDbNull);

        // "NaN" case
        readerMock.Setup(r => r[columnName]).Returns("NaN");
        var resultNaN = (long?)method!.Invoke(null, new object[] { readerMock.Object, columnName });
        Assert.Equal(0, resultNaN);

        // "Infinity" case
        readerMock.Setup(r => r[columnName]).Returns("Infinity");
        var resultInf = (long?)method!.Invoke(null, new object[] { readerMock.Object, columnName });
        Assert.Equal(0, resultInf);
    }

    [Fact]
    public void GetDecimalSafe_ShouldReturnDecimalOrZero()
    {
        // Arrange
        var columnName = _fixture.Create<string>();
        var decimalValue = _fixture.Create<decimal>();

        var readerMock = new Mock<DbDataReader>();
        readerMock.Setup(r => r[columnName]).Returns(decimalValue);

        var method = typeof(CurrentServices).GetMethod(
            "GetDeciamlSafe",
            BindingFlags.NonPublic | BindingFlags.Static);
        Assert.NotNull(method);

        // Act
        var result = (decimal)method!.Invoke(null, new object[] { readerMock.Object, columnName });
        Assert.Equal(decimalValue, result);

        // DBNull case
        readerMock.Setup(r => r[columnName]).Returns(DBNull.Value);
        var resultDefault = (decimal)method!.Invoke(null, new object[] { readerMock.Object, columnName });
        Assert.Equal(0m, resultDefault);
    }
}
