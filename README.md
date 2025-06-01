# LMS2

CREATE PROCEDURE InsertOrderDetails 
    @OrderID INT,
    @ProductID INT,
    @UnitPrice MONEY = NULL,
    @Quantity INT,
    @Discount FLOAT = 0
AS
BEGIN
    DECLARE @AvailableStock INT
    DECLARE @ReorderLevel INT
    DECLARE @ProductUnitPrice MONEY

    -- Get product details
    SELECT @AvailableStock = UnitsInStock,
           @ReorderLevel = ReorderLevel,
           @ProductUnitPrice = UnitPrice
    FROM Products
    WHERE ProductID = @ProductID

    -- Use UnitPrice from table if not provided
    IF @UnitPrice IS NULL
        SET @UnitPrice = @ProductUnitPrice

    -- Check if enough stock is available
    IF @AvailableStock < @Quantity
    BEGIN
        PRINT 'Not enough stock available. Order aborted.'
        RETURN
    END

    -- Insert order details
    INSERT INTO [Order Details](OrderID, ProductID, UnitPrice, Quantity, Discount)
    VALUES (@OrderID, @ProductID, @UnitPrice, @Quantity, @Discount)

    -- Check if insert was successful
    IF @@ROWCOUNT = 0
    BEGIN
        PRINT 'Failed to place the order. Please try again.'
        RETURN
    END

    -- Update stock
    UPDATE Products
    SET UnitsInStock = UnitsInStock - @Quantity
    WHERE ProductID = @ProductID

    -- Check and notify if stock drops below ReorderLevel
    IF (@AvailableStock - @Quantity) < @ReorderLevel
    BEGIN
        PRINT 'Warning: Product stock has fallen below its Reorder Level.'
    END
END

