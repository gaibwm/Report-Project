# Report-Project
 AdventureWork2017 SSRS

Download AdventureWork2017(AdventureWork2017.bak) at https://github.com/Microsoft/sql-server-samples/releases/tag/adventureworks

Below two store-procedures are the only needed to create the report. 
Run this sql command below to create it and from SSRS project create Datasource from. 
It is a simple task for BI developer



CREATE PROCEDURE [dbo].[sp_sales_order_read]
AS
BEGIN
   BEGIN TRY
      SELECT SOH.SalesOrderID, SOH.SalesOrderNumber
      FROM [Sales].[SalesOrderHeader] SOH
         INNER JOIN Sales.Customer C ON SOH.CustomerID = C.CustomerID
         INNER JOIN Sales.Store S ON C.StoreID = S.BusinessEntityID
      ORDER BY  SOH.SalesOrderNumber
   END TRY
   BEGIN CATCH
        DECLARE @ErrorMessage NVARCHAR(4000);
        DECLARE @ErrorSeverity INT;
        DECLARE @ErrorState INT;
        SELECT @ErrorMessage = ERROR_MESSAGE(),
                @ErrorSeverity = ERROR_SEVERITY(),
                @ErrorState = ERROR_STATE();
        RAISERROR (@ErrorMessage, @ErrorSeverity, @ErrorState );
        RETURN 1
    END CATCH
    RETURN 0
END
GO
 
CREATE PROCEDURE [dbo].[sp_sales_order_detail_read]
   @sales_order_id int
AS
BEGIN
   BEGIN TRY
      SELECT   SOD.SalesOrderDetailID, 
               SOD.SalesOrderID,
               SOH.SalesOrderNumber,
               ROW_NUMBER() OVER(ORDER BY SOD.SalesOrderDetailID ASC) AS Line,
               SOD.OrderQty AS Qty,
               P.ProductNumber AS [Item Number],
               P.[Name] AS [Description],
               SOD.CarrierTrackingNumber AS [Tracking #],
               SOD.UnitPrice  AS [Unit Price], 
               (SOD.OrderQty*SOD.UnitPrice) AS Subtotal,
               SOD.LineTotal,
               CASE 
                  WHEN SOD.UnitPriceDiscount IS NULL THEN 0 
                  ELSE 0 - SOD.UnitPrice * SOD.OrderQty * SOD.UnitPriceDiscount
               END AS Discount
      FROM        Sales.SalesOrderDetail SOD INNER JOIN
               Production.Product P ON SOD.ProductID = P.ProductID INNER JOIN
               Sales.SalesOrderHeader SOH ON SOD.SalesOrderID = SOH.SalesOrderID
      WHERE       SOH.SalesOrderID = @sales_order_id
      ORDER BY    SOD.SalesOrderDetailID
   END TRY
   BEGIN CATCH
        DECLARE @ErrorMessage NVARCHAR(4000);
        DECLARE @ErrorSeverity INT;
        DECLARE @ErrorState INT;
        SELECT @ErrorMessage = ERROR_MESSAGE(),
                @ErrorSeverity = ERROR_SEVERITY(),
                @ErrorState = ERROR_STATE();
        RAISERROR (@ErrorMessage, @ErrorSeverity, @ErrorState );
        RETURN 1
    END CATCH
    RETURN 0
END
GO





