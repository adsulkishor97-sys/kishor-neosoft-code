using System;
using System.Collections.Generic;
using System.Reflection;
using AutoFixture;
using Microsoft.VisualStudio.TestTools.UnitTesting;

[TestClass]
public class YourClassTests
{
    private readonly Fixture _fixture = new Fixture();

    [TestMethod]
    public void GetDecimalValue_ShouldReturnDecimal_WhenValueIsNotDBNull()
    {
        // Arrange
        var columnName = _fixture.Create<string>();
        var decimalValue = _fixture.Create<decimal>();

        var fakeReader = new Dictionary<string, object>
        {
            { columnName, decimalValue }
        };

        var method = typeof(YourClassName)
            .GetMethod("GetDecimalValue", BindingFlags.NonPublic | BindingFlags.Static);
        Assert.IsNotNull(method);

        // Act
        var result = method.Invoke(null, new object[] { fakeReader, columnName });

        // Assert
        Assert.AreEqual(decimalValue, result);
    }

    [TestMethod]
    public void GetDecimalValue_ShouldReturnZero_WhenValueIsDBNull()
    {
        // Arrange
        var columnName = _fixture.Create<string>();

        var fakeReader = new Dictionary<string, object>
        {
            { columnName, DBNull.Value }
        };

        var method = typeof(YourClassName)
            .GetMethod("GetDecimalValue", BindingFlags.NonPublic | BindingFlags.Static);
        Assert.IsNotNull(method);

        // Act
        var result = method.Invoke(null, new object[] { fakeReader, columnName });

        // Assert
        Assert.AreEqual(0m, result);
    }
}
