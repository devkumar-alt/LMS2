# LMS2
Q1
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
Q2
CREATE PROCEDURE UpdateOrderDetails
    @OrderID INT,
    @ProductID INT,
    @UnitPrice MONEY = NULL,
    @Quantity INT = NULL,
    @Discount FLOAT = NULL
AS
BEGIN
    DECLARE @OldQuantity INT
    DECLARE @OldUnitPrice MONEY
    DECLARE @OldDiscount FLOAT

    -- Get current values
    SELECT 
        @OldQuantity = Quantity,
        @OldUnitPrice = UnitPrice,
        @OldDiscount = Discount
    FROM [Order Details]
    WHERE OrderID = @OrderID AND ProductID = @ProductID

    IF @@ROWCOUNT = 0
    BEGIN
        PRINT 'Order detail not found. Cannot update.'
        RETURN
    END

    DECLARE @FinalQuantity INT = ISNULL(@Quantity, @OldQuantity)
    DECLARE @FinalUnitPrice MONEY = ISNULL(@UnitPrice, @OldUnitPrice)
    DECLARE @FinalDiscount FLOAT = ISNULL(@Discount, @OldDiscount)

    -- Adjust stock: Add back old quantity, subtract new quantity
    UPDATE Products
    SET UnitsInStock = UnitsInStock + @OldQuantity - @FinalQuantity
    WHERE ProductID = @ProductID

    -- Update order details
    UPDATE [Order Details]
    SET 
        UnitPrice = @FinalUnitPrice,
        Quantity = @FinalQuantity,
        Discount = @FinalDiscount
    WHERE OrderID = @OrderID AND ProductID = @ProductID

    IF @@ROWCOUNT > 0
        PRINT 'Order details updated successfully.'
    ELSE
        PRINT 'Failed to update order details.'
END
Q3
CREATE PROCEDURE GetOrderDetails
    @OrderID INT
AS
BEGIN
    IF NOT EXISTS (
        SELECT 1 FROM [Order Details] WHERE OrderID = @OrderID
    )
    BEGIN
        PRINT 'The OrderID ' + CAST(@OrderID AS VARCHAR) + ' does not exist';
        RETURN 1;
    END

    SELECT * 
    FROM [Order Details]
    WHERE OrderID = @OrderID;
END
Q4
CREATE PROCEDURE DeleteOrderDetails
    @OrderID INT,
    @ProductID INT
AS
BEGIN
    IF NOT EXISTS (
        SELECT 1 FROM [Order Details]
        WHERE OrderID = @OrderID AND ProductID = @ProductID
    )
    BEGIN
        PRINT 'Invalid OrderID or ProductID. Deletion failed.';
        RETURN -1;
    END

    DELETE FROM [Order Details]
    WHERE OrderID = @OrderID AND ProductID = @ProductID;

    PRINT 'Order detail deleted successfully.';
END

