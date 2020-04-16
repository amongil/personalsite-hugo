---
title: "Terralib: Run Terraform commands in your Go code"
date: 2020-04-16T12:13:07+02:00
draft: false
toc: false
images:
tags: 
  - terraform
  - go
  - library
---

![Terraform logo](https://avatars0.githubusercontent.com/u/28900900?s=460&v=4)
![Gopher](https://blog.golang.org/gopher/header.jpg)

When developing automated infrastructure tests, I've often had to use Terraform to spin up an ephemeral cloud environment and then perform some tests on it, using the AWS or the Azure SDK (i.e. checking connectivity or next hops with Azure Network Watcher API calls).

Terraform is only supported through a Command-Line-Interface, and I wanted to perform all operations in a single Go program.

That's the reason I started to dig into the Terraform code a create a library of my own so I could call Terraform CLI commands within my Go programs. I have just released [working alpha of my Terralib project](https://github.com/amongil/terralib).

Example of usage: 

```Go
package main

import (
	"fmt"
	"log"
	"os"

	terralib "github.com/amongil/terralib"
)

func main() {
    // Init terralib. ConfigPath is the path on where the Terraform config files are
	tf := terralib.Terralib{
		ConfigPath: "terraform-files",
    }
    
    // Terraform init
	log.Println("Running terraform init...")
	options := []string{}
	initOutput, err := tf.Init(options)
	if err != nil {
		defer fmt.Println(initOutput.Raw)
		log.Printf("Error on terraform init: %s\n", err)
		// We can act on specific errors and maybe remediate
		if err.Error() == terralib.ErrProviderNotFound {
			log.Printf("Full error: %s\n", err.(terralib.InitError).Reason)
		}
		return
    }
    
    // Terraform plan
	log.Println("Running terraform plan...")
	planOptions := []string{
		"-out=planfile",
	}
	_, err = tf.Plan(planOptions)
	if err != nil {
		log.Printf("Error on terraform plan: %s\n", err)
		if err.Error() == terralib.ErrInvalidResourceType {
			log.Printf("Full error: %s\n", err.(terralib.PlanError).Reason)
		}
		return
    }
    
    // Issue Terraform Show command on the plan file and save info on memory
	dir, err := os.Getwd()
	if err != nil {
		log.Fatal(err)
	}
	showOutput, err := tf.Show(dir + "/terraform-files/planfile")
	// Having our planned resource changes in a map allows us to make decisions over them
	// maybe even send them to event hubs for posterior analysis?
	plannedValues := (showOutput.PlannedValues).(map[string]interface{})
	for k, v := range plannedValues {
		fmt.Printf("%s: %v\n", k, v.(map[string]interface{})["resources"])
	}

    // Terraform apply
	log.Println("Running terraform apply...")
	applyOptions := []string{
		"-auto-approve",
	}
	applyOutput, err := tf.Apply(applyOptions)
	if err != nil {
		log.Printf("Error on terraform apply: %s\n", err)
		return
	}
	fmt.Println(applyOutput)

}
```
 
See how a complete Terraform workflow can be executed within your Go program. It needs more tests, error handling and deduplication of code, but it already serves a purpose. Hope you enjoy the idea and you start contributing to it. Don't hesitate to get in touch via GitHub.