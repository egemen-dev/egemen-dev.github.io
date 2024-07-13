---
layout: post
title: "What causes double render and how to prevent it?"
date: 2024-03-01 20:00:00 +0300
---

# What causes double render and how to prevent it?

In ruby on rails backend development, developers might face a common situation captured in the following code snippet:

```rb
def action
  check_params

  render json: { message: "Action method worked, it's all good!" }, status: 200
end

def check_params
  if params[:important_param] == ""
    render json: { error: "Important param is missing" }, status: 404
    
    return
  end
end
```

## Purpose
Basically, code is set up in a way that check the input params before the `action` method and avoid certain situations by stopping the execution right after the condition inside the `check_params` method is met. However, this is not happening and you get  `AbstractController::DoubleRenderError`. The problem is that the `check_params` method is not stoppig the execution with 404 error response.

## Breaking down the problem
The reason for the double rendering error here is because the `check_params` method's return statement is only exiting from the current method which is itself and not the entire `action` so that it reaches to the bottom of the `action` method and renders whatever is set to render, and you get yourself a nice double render error `AbstractController::DoubleRenderError`. This error indicates that there are multiple renders called in this particular action.

## Solution

`before_action` ✅

Official Rails [doc](https://api.rubyonrails.org/v7.1.3.2/classes/AbstractController/Callbacks/ClassMethods.html#method-i-before_action) says about the `before_action`:
> Append a callback before actions. If the callback **renders** or redirects, the action _will not run_. If there are additional callbacks scheduled to run after that callback, **they are also cancelled**.

This is the exact beheviour the code should have.

Lets's refactor the code in order to utilize the before_action callback:
```rb
before_action :check_params, only: :action

def action
  render json: { message: "Action method worked, it's all good!" }, status: 200
end

def check_params
  if params[:important_param] == ""
    render json: { error: "Important param is missing" }, status: 404
    
    return
  end
end
```
