using System;
using System.Data.Common;
using System.Globalization;
using System.Reflection;
using Xunit;
using AutoFixture;

public class AssetServiceTests
{
    private readonly Type _serviceType = typeof(AssetService);
    private readonly Fixture _fixture = new();

    private DbDataReader CreateMockReader(string columnName, object value)
    {
        var table = new System.Data.DataTable();
        table.Columns.Add(columnName, typeof(object));
        var row = table.NewRow();
        row[columnName] = value;
        table.Rows.Add(row);
        return table.CreateDataReader();
    }

    [Fact]
    public void GetStringValue_ShouldReturnExpectedString_WhenValueExists()
    {
        // Arrange
        string columnName = _fixture.Create<string>().Replace(" ", "");
        string expectedValue = _fixture.Create<string>();
        var reader = CreateMockReader(columnName, expectedValue);
        reader.Read();

        var method = _serviceType.GetMethod("GetStringValue", BindingFlags.NonPublic | BindingFlags.Static);

        // Act
        var result = method!.Invoke(null, new object[] { reader, columnName });

        // Assert
        Assert.Equal(expectedValue, result);
    }

    [Fact]
    public void GetStringValue_ShouldReturnDefault_WhenValueIsDBNull()
    {
        string columnName = _fixture.Create<string>().Replace(" ", "");
        var reader = CreateMockReader(columnName, DBNull.Value);
        reader.Read();

        var method = _serviceType.GetMethod("GetStringValue", BindingFlags.NonPublic | BindingFlags.Static);
        var result = method!.Invoke(null, new object[] { reader, columnName });

        Assert.Equal("0", result);
    }

    [Fact]
    public void GetDecimalValue_ShouldReturnExpectedDecimal_WhenValueExists()
    {
        string columnName = _fixture.Create<string>().Replace(" ", "");
        decimal expectedValue = _fixture.Create<decimal>();
        var reader = CreateMockReader(columnName, expectedValue);
        reader.Read();

        var method = _serviceType.GetMethod("GetDecimalValue", BindingFlags.NonPublic | BindingFlags.Static);
        var result = method!.Invoke(null, new object[] { reader, columnName });

        Assert.Equal(expectedValue, (decimal?)result);
    }

    [Fact]
    public void GetDecimalValue_ShouldReturnZero_WhenValueIsDBNull()
    {
        string columnName = _fixture.Create<string>().Replace(" ", "");
        var reader = CreateMockReader(columnName, DBNull.Value);
        reader.Read();

        var method = _serviceType.GetMethod("GetDecimalValue", BindingFlags.NonPublic | BindingFlags.Static);
        var result = method!.Invoke(null, new object[] { reader, columnName });

        Assert.Equal(0m, (decimal?)result);
    }

    [Fact]
    public void GetUnixTime_ShouldReturnExpectedUnixTime_WhenValidDateProvided()
    {
        string columnName = _fixture.Create<string>().Replace(" ", "");
        string dateValue = _fixture.Create<DateTime>().ToString("yyyyMMdd", CultureInfo.InvariantCulture);
        var reader = CreateMockReader(columnName, dateValue);
        reader.Read();

        var method = _serviceType.GetMethod("GetUnixTime", BindingFlags.NonPublic | BindingFlags.Static);
        var result = method!.Invoke(null, new object[] { reader, columnName });

        var expectedUnix = new DateTimeOffset(DateTime.ParseExact(dateValue, "yyyyMMdd", CultureInfo.InvariantCulture), TimeSpan.Zero)
            .ToUnixTimeMilliseconds();

        Assert.Equal(expectedUnix, (long?)result);
    }

    [Fact]
    public void GetUnixTime_ShouldReturnZero_WhenValueIsDBNull()
    {
        string columnName = _fixture.Create<string>().Replace(" ", "");
        var reader = CreateMockReader(columnName, DBNull.Value);
        reader.Read();

        var method = _serviceType.GetMethod("GetUnixTime", BindingFlags.NonPublic | BindingFlags.Static);
        var result = method!.Invoke(null, new object[] { reader, columnName });

        Assert.Equal(0L, (long?)result);
    }
}
