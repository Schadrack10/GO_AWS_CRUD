# GO_AWS_CRUD
Hii im shakes! , if you wish you become a golang pro ,follow these steps ..Good luck!!


Golang CRUD application intergrated with aws dynamo DB

# First, make sure you have the aws-sdk-go package installed:

    go get github.com/aws/aws-sdk-go/aws
    go get github.com/aws/aws-sdk-go/aws/session
    go get github.com/aws/aws-sdk-go/service/dynamodb

# main script

    package main
    
    import (
    	"fmt"
    	"os"
    	"github.com/aws/aws-sdk-go/aws"
    	"github.com/aws/aws-sdk-go/aws/session"
    	"github.com/aws/aws-sdk-go/service/dynamodb"
    	"github.com/aws/aws-sdk-go/service/dynamodb/dynamodbattribute"
    )
    
    var tableName = "recipes"
    
  # Recipe represents the structure of a recipe item in DynamoDB
  
    type Recipe struct {
    	RecipeShop   string
    	RecipeID     string
    	RecipeAuthor string
    	RecipeName   string
    }
    
  # Create Recipe
  
    func createRecipe(db *dynamodb.DynamoDB, recipe Recipe) error {
    	av, err := dynamodbattribute.MarshalMap(recipe)
    	if err != nil {
    		return err
    	}
    
    	input := &dynamodb.PutItemInput{
    		Item:      av,
    		TableName: aws.String(tableName),
    	}
    
    	_, err = db.PutItem(input)
    	return err
    }

 # Get Recipe   
    
    func getRecipe(db *dynamodb.DynamoDB, recipeShop, recipeID string) (*Recipe, error) {
    	input := &dynamodb.GetItemInput{
    		Key: map[string]*dynamodb.AttributeValue{
    			"RecipeShop": {S: aws.String(recipeShop)},
    			"RecipeID":   {S: aws.String(recipeID)},
    		},
    		TableName: aws.String(tableName),
    	}
    
    	result, err := db.GetItem(input)
    	if err != nil {
    		return nil, err
    	}
    
    	if result.Item == nil {
    		return nil, nil // Recipe not found
    	}
    
    	recipe := new(Recipe)
    	err = dynamodbattribute.UnmarshalMap(result.Item, &recipe)
    	return recipe, err
    }

# Update Recipe
    
    func updateRecipe(db *dynamodb.DynamoDB, recipe Recipe) error {
    	av, err := dynamodbattribute.MarshalMap(recipe)
    	if err != nil {
    		return err
    	}
    
    	input := &dynamodb.UpdateItemInput{
    		Key: map[string]*dynamodb.AttributeValue{
    			"RecipeShop": {S: aws.String(recipe.RecipeShop)},
    			"RecipeID":   {S: aws.String(recipe.RecipeID)},
    		},
    		ExpressionAttributeValues: av,
    		TableName:                 aws.String(tableName),
    		ReturnValues:              aws.String("UPDATED_NEW"),
    		UpdateExpression:          aws.String("SET RecipeAuthor = :ra, RecipeName = :rn"),
    	}
    
    	_, err = db.UpdateItem(input)
    	return err
    }

 # Delete Recipe   
    
    func deleteRecipe(db *dynamodb.DynamoDB, recipeShop, recipeID string) error {
    	input := &dynamodb.DeleteItemInput{
    		Key: map[string]*dynamodb.AttributeValue{
    			"RecipeShop": {S: aws.String(recipeShop)},
    			"RecipeID":   {S: aws.String(recipeID)},
    		},
    		TableName: aws.String(tableName),
    	}
    
    	_, err := db.DeleteItem(input)
    	return err
    }

 # main function 
    
    func main() {

 # Replace these values with your AWS credentials and region

    	awsAccessKey := "your_access_key"
    	awsSecretKey := "your_secret_key"
    	awsRegion := "your_region"
    
    	sess, err := session.NewSession(&aws.Config{
    		Region:      aws.String(awsRegion),
    		Credentials: credentials.NewStaticCredentials(awsAccessKey, awsSecretKey, ""),
    	})
    	if err != nil {
    		fmt.Println("Error creating session:", err)
    		return
    	}
    
    	db := dynamodb.New(sess)
    
  # Example: Create Recipe
  
    	newRecipe := Recipe{
    		RecipeShop:   "exampleShop",
    		RecipeID:     "exampleID",
    		RecipeAuthor: "John Doe",
    		RecipeName:   "Chicken Curry",
    	}
    
    	err = createRecipe(db, newRecipe)
    	if err != nil {
    		fmt.Println("Error creating recipe:", err)
    		return
    	}
    	fmt.Println("Recipe created successfully!")
    
  # Example: Get Recipe
  
    	retrievedRecipe, err := getRecipe(db, "exampleShop", "exampleID")
    	if err != nil {
    		fmt.Println("Error getting recipe:", err)
    		return
    	}
    	if retrievedRecipe != nil {
    		fmt.Println("Retrieved Recipe:", retrievedRecipe)
    	} else {
    		fmt.Println("Recipe not found.")
    	}
    
  # Example: Update Recipe
  
    	updatedRecipe := Recipe{
    		RecipeShop:   "exampleShop",
    		RecipeID:     "exampleID",
    		RecipeAuthor: "Jane Doe",
    		RecipeName:   "Spaghetti Bolognese",
    	}
    
    	err = updateRecipe(db, updatedRecipe)
    	if err != nil {
    		fmt.Println("Error updating recipe:", err)
    		return
    	}
    	fmt.Println("Recipe updated successfully!")
    
  #  Example: Delete Recipe
  
    	err = deleteRecipe(db, "exampleShop", "exampleID")
    	if err != nil {
    		fmt.Println("Error deleting recipe:", err)
    		return
    	}
    	fmt.Println("Recipe deleted successfully!")
    }
