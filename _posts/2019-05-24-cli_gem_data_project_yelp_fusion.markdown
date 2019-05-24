---
layout: post
title:      "CLI Gem Data Project: Yelp Fusion"
date:       2019-05-24 20:58:57 +0000
permalink:  cli_gem_data_project_yelp_fusion
---


I am new to using APIs and I chose the Yelp API to look up restaurants from Ventura. 

What follows is my process that I went through to get that data.




## API class

### Authentication

First up, you need the "HTTParty" gem to make requests.
 
Yelp requires a "Client ID" or "APIKey" to connect to their API.

https://www.yelp.com/fusion  (Go here to get your Client ID or APIkey. You will need to create a user account if you dont already have one.)

**Keep your Key Secret**

You dont want to broadcast your Key and ID to all of the internet so were going to need a .ENV file.

1. You will need the "dotenv" gem
          - Add the dotenv gem to your gemfile and run a `bundle install`
          - Require dotenv in the same place you required things like pry, HTTParty etc 

2. Create a .ENV file in the root of your project folder.

3. In the .ENV file you put your ClientID or APIKey.
             -(You only need one or the other not both)

             Example of .ENV file:
             ```
            #Your key or ID go where the Xs are
             YELP_API_KEY=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

             YELP_CLIENT_ID=XXXXXXXXXXXX
             ```
 

  Add this to your HTTP request :
```
        {headers: {"Authorization" => "Bearer #{ENV['YELP_API_KEY']}"}})
```

**Your final HTTP request should look something like this:**

```
HTTParty.get("https://api.yelp.com/v3/businesses/search?term=Restaurants&location=Ventura", {headers: {"Authorization" => "Bearer #{ENV['YELP_API_KEY']}"}})
```
					
### Making HTTP requests

Now that we are authenticated we can get our information from yelp.

This is the most basic and broadest request that you can make:

```
https://api.yelp.com/v3/businesses/search

```
From here we can add different search crteria to narrow down our search.
         
**Yelp has a list of different search terms that it accepts here:**
https://www.yelp.com/developers/documentation/v3/business_search
						
In my CLI project, I was looking for restaurants in Ventura.
So I searched for the term "restaurants" and then set the location to Ventura.


## Restaurant Class 

At this point I was ready to create objects from the data I acquired from Yelp

The data that is returned from our HTTPrequest comes in the form of a hash.

I had an attr_accessor for every key in the hash and then set the values of those attrs to match the values of the keys in the hash.

```
class VenturaRestaurants::Restaurant
    attr_accessor :id, :alias, :name, :image_url, :is_closed, :url, :review_count, :categories, :rating, :coordinates, :transactions, :price, :location, :phone, :display_phone, :distance

    @@all = []

    def initialize(restaurant)
        restaurant_from_hash(restaurant)
        save
    end

    def save 
        @@all << self
      end
       
    def self.all
        @@all
    end

    def self.new_from_collection(restaurants)
        restaurants.each do |restaurant|
            new(restaurant)
        end 
    end

#Here I am iterating through the hash and assigning values to my attr_accessors from the values of the keys.

    def restaurant_from_hash(restaurant)
        restaurant.each do |key, value|
            send("#{key}=", value)
        end
        
    end

end    
```



## CLI Class

### Selecting Restaurant from a category

Initially I wanted the user to see a list of all the restaurants in Ventura or see a list of genres of food in Ventura and then choose a genre from there.

```
   def list_categories
        categories = []
        VenturaRestaurants::Restaurant.all.each do |restaurant|
             categories << restaurant.categories[0]["title"]
            end
        categories.uniq!.each do |category|
            puts "#{@@green}#{category}"
        end           
    end
```


In this method they were able to select a category by typing the name of the category.

I used regex to match the user input to a category in the list.

Once the category was selected it would list all the restaurants in the category
```
def select_category_and_list_restaurants_in_category      
        @restaurants_in_category = []
        VenturaRestaurants::Restaurant.all.each do |restaurant|      
            @restaurants_in_category << restaurant if restaurant.categories[0]["title"] =~ /#{@input}/
            end
            @restaurants_in_category.each_with_index do |restaurant, index|
            puts "#{@@green}#{index + 1}. #{restaurant.name}"
        end           
    end
```

Here they select the restaurant from the list
```
def select_restaurant_in_category
       @restaurant = @restaurants_in_category[@input]
    end
```		

Once they select a restaurant, they are able to see the details of that restaurant.
```
    def display_restaurant_details
        puts ""
        puts "    #{@restaurant.name}"
        puts ""
        puts "Phone:#{@restaurant.display_phone}"
        puts "Price:#{@restaurant.price}"
        puts "Rating:#{@restaurant.rating}"
        puts "Review Count:#{@restaurant.review_count}"
        puts "Yelp Page:#{@restaurant.url}"
        puts "Is Closed?:#{@restaurant.is_closed}"
        puts ""
    end
```

###  Display All restaurants and then select a restaurant

```
 #This displays a numbered list of all restaurants in Ventura
    def list_restaurants
        VenturaRestaurants::Restaurant.all.each_with_index do |restaurant, index|
            puts "#{@@green}#{index + 1}. #{restaurant.name}"
        end 
    end

#This allows the user to select a restaurant 
    def select_restaurant   
        @restaurant = VenturaRestaurants::Restaurant.all[@input]
    end
		
	#This selects ALL the restaurants and displays the details of each restaurant.	
		   def select_and_display_all_restaurants
        VenturaRestaurants::Restaurant.all.each do |restaurant| 
            @restaurant = restaurant
            display_restaurant_details
        end
    end
		
		
```


### Validating User Input

However I needed loops and statements to validate all of this user input. 

Here it is:


This "while" loop allowed the user to come back to the beginning.
```
        while @input != "N"
            puts "Type 'genre' to view Restaurants by genre and type 'all' to view all the restaurants!"
            get_user_input
```						
       The "case" statement is saying when @input == "string" then do stuff.
``` 
            case @input
						
			#This is the first "when" of my "case" statement.	
            when "genre"
                list_categories
                puts "Type in the name of the Genre(or part of the name)"
                get_user_input
                select_category_and_list_restaurants_in_category
                puts "Select the number of the Restaurant OR type 'all' OR type 'back' to go back to the top"
                get_user_input
```							
			    If the user types "all" they can view the details of all the restaurants.
```
                if @input == "all"
                    select_and_display_all_restaurants_in_category          
```									
			    If the user types "back" then they can go back to the beginning.
```
                elsif @input != "back"
                    input_to_index
                    select_restaurant_in_category
                    display_restaurant_details
                end 
```	
	
		  This is the second "when" of my "case" statement.	
```
            when "all"
                list_restaurants
                puts "Select the number of the Restaurant OR type 'all' OR type 'back' to go back to the top"
                get_user_input
```							
				Here they can view all restaurants and their details	
```
                if @input == "all"
                    select_and_display_all_restaurants
```						
				Here if they type "back" then they can go back to the top		
```
                elsif @input != "back"   
                    input_to_index 
                    select_restaurant
                    display_restaurant_details
                end
            end
```							    
				Here it allows the user to view another restaurant if they want to. 	

```
           if @input != "back"
                puts "Do you want to see another restaurant?"
                get_user_input
            end
        end  
    end
```		


That was pretty much it. You really could use this code with anything you want from Yelp. 

You could do Zoos, Aquariums and even parks. 

I was just demonstrating how to access the Yelp API and build a CLI interface.



