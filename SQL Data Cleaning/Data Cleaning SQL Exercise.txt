-- DATA CLEANING SQL

-- Standardizing Dates
UPDATE nashville_houses
Set SaleDateConverted = CONVERT(Date, SaleDate)

SELECT SaleDateConverted
FROM nashville_houses

-- Populate Null propertyaddresses

SELECT a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress, ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM nashville_houses a
JOIN nashville_houses b
	ON a.ParcelID = b.ParcelID
	AND a.[UniqueID ] != b.[UniqueID ]
WHERE a.PropertyAddress IS NULL


UPDATE a
SET PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM nashville_houses a
JOIN nashville_houses b
	ON a.ParcelID = b.ParcelID
	AND a.[UniqueID ] != b.[UniqueID ]
WHERE a.PropertyAddress IS NULL

-- Splitting up address into separate columns

SELECT PropertyAddress
FROM nashville_houses

SELECT SUBSTRING(PropertyAddress, 1, CHARINDEX(',',PropertyAddress) -1 ) AS Address,
SUBSTRING(PropertyAddress, CHARINDEX(',',PropertyAddress) +1, LEN(PropertyAddress)) AS Address1
FROM nashville_houses

ALTER TABLE nashville_houses
ADD PropertySplitAddress NVARCHAR(255)

UPDATE nashville_houses
SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',',PropertyAddress) -1 )

ALTER TABLE nashville_houses
ADD PropertySplitCity NVARCHAR(255)

UPDATE nashville_houses
SET PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',',PropertyAddress) +1, LEN(PropertyAddress))

-- Owner Address Split

SELECT OwnerAddress
FROM nashville_houses

SELECT PARSENAME(REPLACE(OwnerAddress, ',', '.'),3) AS Address,
PARSENAME(REPLACE(OwnerAddress, ',', '.'),2) AS City,
PARSENAME(REPLACE(OwnerAddress, ',', '.'),1) AS State
FROM nashville_houses

ALTER TABLE nashville_houses
ADD OwnerSplitAddress NVARCHAR(255);

UPDATE nashville_houses
SET OwnerSplitAddress = PARSENAME(REPLACE(OwnerAddress, ',', '.'),3)

ALTER TABLE nashville_houses
ADD OwnerSplitCity NVARCHAR(255);

UPDATE nashville_houses
SET OwnerSplitCity = PARSENAME(REPLACE(OwnerAddress, ',', '.'),2)

ALTER TABLE nashville_houses
ADD OwnerSplitState NVARCHAR(255);

UPDATE nashville_houses
SET OwnerSplitState = PARSENAME(REPLACE(OwnerAddress, ',', '.'),1)

-- Changing Y to Yes and N to No

SELECT DISTINCT(SoldAsVacant), COUNT(SoldAsVacant)
FROM nashville_houses
GROUP BY SoldAsVacant
ORDER BY 2


SELECT SoldAsVacant,
	CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
	WHEN SoldAsVacant = 'N' THEN 'No'
	ELSE SoldAsVacant
	END
FROM nashville_houses

UPDATE nashville_houses
SET SoldAsVacant = CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
						WHEN SoldAsVacant = 'N' THEN 'No'
						ELSE SoldAsVacant
					END

-- Remove Duplicates

WITH row_num_cte AS(
SELECT *, 
	ROW_NUMBER() OVER(
	PARTITION BY ParcelID,
				PropertyAddress, 
				SalePrice,
				SaleDate,
				LegalReference
				ORDER BY 
					UniqueID
					) AS Row_num
FROM nashville_houses
)
DELETE
FROM row_num_cte
WHERE row_num > 1

-- Delete Unused Columns

SELECT *
FROM nashville_houses

ALTER TABLE nashville_houses
DROP COLUMN PropertyAddress, OwnerAddress, TaxDistrict, SaleDate