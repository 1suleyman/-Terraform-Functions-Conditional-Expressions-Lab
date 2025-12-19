# 🧩 Terraform Functions & Conditional Expressions Lab

In this lab, I learned how to **use Terraform functions, meta-arguments, and conditional expressions** to transform values, dynamically create resources, and make configuration decisions based on variables — using both **Terraform console** and real configuration files.

---

## 📋 Lab Overview

**Goal:**

* Explore Terraform built-in functions using `terraform console`
* Understand how functions transform values
* Convert variable types without modifying the original variable
* Dynamically create multiple AWS resources using `count` and `for_each`
* Use conditional expressions to control resource behavior

**Learning Outcomes:**

* Use Terraform console to test functions interactively
* Apply string, numeric, and collection functions
* Convert strings → lists using `split()`
* Create multiple IAM users using `count`
* Upload multiple S3 objects using `for_each`
* Use conditional expressions to select instance types dynamically

---

## 🛠 Step-by-Step Journey

### Step 1: Explore Terraform Functions with Console

**Task:** Use Terraform console to test how functions transform values.

#### Numeric Function

```hcl
floor(10.9)
```

* Output: `10`
* `floor()` rounds a number **down to the nearest whole number**

---

#### String Function

```hcl
title("user-generated password file")
```

* Output: `User-Generated Password File`
* `title()` capitalizes the **first letter of each word**

---

#### Collection Function

**Question:** Which type does `lookup()` work with?

* ✅ Answer: **Maps**

---

### Step 2: Inspect Variable Types

Navigate to:

```
/root/terraform-projects/project-sonic
```

**Question:** What type is `cloud_users`?

```hcl
type = string
```

* Even though it holds multiple values, it is stored as **a single string**

---

### Step 3: Convert String → List (Without Changing variables.tf)

**Task:**
Create IAM users for each developer name stored in a colon-separated string.

Example variable value:

```
mario:luigi:sonic:tails:knuckles:amy:braja
```

---

### Step 4: Create IAM Users Using `count`

**Resource:**

```hcl
resource "aws_iam_user" "cloud" {
  name  = split(":", var.cloud_users)[count.index]
  count = length(split(":", var.cloud_users))
}
```

**Explanation:**

* `split()` converts the string into a list
* `count` creates one user per list element
* `count.index` selects the correct name

---

### Step 5: Apply the Configuration

```bash
terraform init
terraform plan
terraform apply
```

✔️ Multiple IAM users are created dynamically

---

### Step 6: Query Resource Values Using Terraform Console

**Question:**
What is the name of the IAM user at index `6`?

```bash
echo 'aws_iam_user.cloud[6].name' | terraform console
```

* Output: `braja`

---

### Step 7: Find Index of an Element in a List

**Task:** Find the index of `"oni"` in the variable `sf`.

```bash
echo 'index(var.sf, "oni")' | terraform console
```

* Output: `7`

---

### Step 8: Upload Multiple Files to S3 Using `for_each`

**Task:**
Upload media files to an S3 bucket using `for_each`.

---

#### Resource: Upload Media Files

```hcl
resource "aws_s3_object" "upload_sonic_media" {
  for_each = var.media

  bucket = aws_s3_bucket.sonic_media.id
  source = each.value
  key    = substr(each.value, 7, 20)
}
```

**Explanation:**

* `for_each` loops over each file
* `each.value` represents the current file path
* `substr()` removes the `sonic-media/` prefix from the filename

---

### Step 9: Apply S3 Upload Configuration

```bash
terraform apply
```

✔️ All media files uploaded successfully

---

### Step 10: Conditional Expressions (EC2 Instance)

Navigate to:

```
/root/terraform-projects/project-mario
```

---

#### Variable Inspection

| Variable | Value       |
| -------- | ----------- |
| small    | `t2.nano`   |
| name     | `undefined` |

---

### Step 11: Create EC2 Instance with Conditional Logic

**Resource:**

```hcl
resource "aws_instance" "mario_server" {
  ami           = var.ami
  instance_type = var.name == "tiny" ? var.small : var.large

  tags = {
    Name = var.name
  }
}
```

**Explanation:**

* If `name == "tiny"` → use `small`
* Otherwise → use `large`
* Conditional syntax:
  `condition ? true_value : false_value`

---

## ✅ Key Commands Summary

| Task                      | Command / Function            |
| ------------------------- | ----------------------------- |
| Open Terraform console    | `terraform console`           |
| Round number down         | `floor(10.9)`                 |
| Capitalize words          | `title("text")`               |
| Convert string to list    | `split(":", var.cloud_users)` |
| Create multiple resources | `count`                       |
| Loop over collections     | `for_each`                    |
| Find index in list        | `index(var.sf, "oni")`        |
| Conditional logic         | `condition ? true : false`    |

---

## 💡 Notes / Tips

* Terraform console is **safe** — it doesn’t change infrastructure
* `count` works best with **lists**
* `for_each` works best with **maps and sets**
* Use `split()` to avoid changing original variable definitions
* Conditional expressions are powerful for **environment-aware configs**

---

## 📌 Lab Summary

| Area                          | Result | Key Takeaway                     |
| ----------------------------- | ------ | -------------------------------- |
| Terraform console usage       | ✅      | Tested functions interactively   |
| String & numeric functions    | ✅      | `floor`, `title`, `split`        |
| Dynamic IAM user creation     | ✅      | Used `count` + `split()`         |
| S3 object uploads             | ✅      | Used `for_each` with substrings  |
| Conditional EC2 configuration | ✅      | Instance size chosen dynamically |

---

## ✅ References

* [https://developer.hashicorp.com/terraform/language/functions](https://developer.hashicorp.com/terraform/language/functions)
* [https://developer.hashicorp.com/terraform/language/expressions/conditionals](https://developer.hashicorp.com/terraform/language/expressions/conditionals)
* [https://developer.hashicorp.com/terraform/cli/commands/console](https://developer.hashicorp.com/terraform/cli/commands/console)
* [https://registry.terraform.io/providers/hashicorp/aws/latest/docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
