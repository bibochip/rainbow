## Issue-35: Update User Role

### Default user registration is customer, admin user can update all user's role

We need to create a form for admin to update all user's role, a new user registrate will be set to customer role (rule have set when create a new one).

What we need:
 * Update routes to have resources :users
 * create controllers/users_controller.rb for updating user role
 * create views/users folder to contain forms for updating user role
 * expect an admin_user can be able to update all users role

# Routes

### update file routes: config/routes.rb
```ruby
Rails.application.routes.draw do
  root 'home#index'
  resources :products
  devise_for :users
  resources :categories
  resources :lunch_orders, only: [:index, :show]
  resources :menus
  resources :product_imports do
    collection { post :import }
  end
  resources :users
end
```

# Controllers

### Create a file with name: **app/controllers/users_controller.rb**
```ruby
class UsersController < ApplicationController
  load_and_authorize_resource

  def index
    show list of users order(created_at: :desc)
  end

  def edit
    show form for updating user with user information filled
  end

  def show
    show info about a user
  end

  def update
    updated user_role? user_detail : render 'edit'
  end

  def destroy
    render 'index' if destroy user successfully 
  end

  private
    def user_find
      User.find(params[:id])
    end
end
```

# Views

### Update file: **app/views/shared/_nav.html.haml**
```ruby
%nav.navbar.navbar-default
  .container
    .navbar-header
      %h1=app_name
    %ul.nav.navbar-nav.navbar-right
      %li= link_to 'Sign Up', new_user_registration_path
      %li= link_to 'Log In', new_user_session_path
      - if can? :manage, User
        %li= link_to 'User Manage', users_path
```

### Create a file with name: **app/views/users/index.html.haml**
```ruby
%h2 List of Users
%hr
.panel.panel-default
%table.table.table-responsive.table-striped
  %tr
    %th Email
    %th Role
    %th
  - @users.each do |user|
    %tr
      %td= user.email
      %td= user.role
      %td.text-right
        - if can? :update, User
          = link_to 'Edit', edit_user_path(user), class: 'btn btn-primary'
        - if can? :read, User
          = link_to 'Show', user_path(user), class: 'btn btn-info'
        - if can? :destroy, User
          = link_to 'Destroy', user_path(user), method: :delete, data: { confirm: 'Are you sure?' }, class: 'btn btn-danger'
```

### Create a file with name: **app/views/users/edit.html.haml**
```ruby
.col-md-6.col-md-offset-3
  .edit-book-form
    %h1 Edit Product
    = simple_form_for @user, html: { multipart: true } do |f|
      = f.input :email, readonly: true
      = f.input :role_user, collection: user.role_users
      = f.button :submit
```

### Create a file with name: **app/views/users/show.html.haml**
```ruby
.col-md-8
  .links.btn-group
  - if can? :read, User
    = link_to 'Back To Index', users_path, class: 'btn btn-custom'
    - if can? :update, User
      = link_to 'Edit', edit_user_path(@user) , class: 'btn btn-custom'
    - if can? :destroy, User  
      = link_to 'Delete', user_path(@user), method: :delete, data: {confirm: 'Are you sure?'} , class: 'btn btn-custom'
  .book-info
    %table.table
      %tr
        %td
          Email: 
        %td
          = @user.email
      %tr
        %td
          Role: 
        %td
          = @user.role_user
      %tr
        %td
          Created At: 
        %td
          = @user.created_at
      %tr
        %td
          Updated at: 
        %td
          = @user.updated_at
```

# Spec

### Create a file with name: **spec/controllers/users_controller_spec.rb**
```ruby
require 'rails_helper'

RSpec.describe UsersController, type: :controller do
  let!(:user_admin) { create(:user, role_user: :admin) }
  let!(:user_1) { create(:user, role_user: :restaurant) }
  let!(:user_2) { create(:user, role_user: :customer) }

  describe 'GET index' do
    before { get :index }

    specify do
      expect(assigns(:users)).to eq [user_2, user_1]
      expect(response).to have_http_status(200)
      expect(response).to render_template(:index)
    end
  end

  describe 'GET show' do
    specify do
      get :show, params: { id: user_1.id }
      expect(response).to render_template(:show)
      expect(assigns(:user)).to eq user_1
    end
  end

  describe 'GET edit' do
    specify do
      get :edit, params: { id: user_1.id }
      expect(response).to render_template(:edit)
    end
  end

  describe 'PATCH update' do
    let(:params) do
      {
        role_user: :customer
      }
    end

    context 'success' do
      before {
        patch :update, params: { user: params, id: user_1.id }
      }

      specify do
        user_1.reload
        expect(user_1.role_user).to eq(:customer)
        expect(response).to redirect_to(users_path)
      end
    end

    context 'failure' do 
      before {
        allow_any_instance_of(User).to receive(:update).and_return(false)
      }

      specify do
        expect do
          patch :update, params: { user: params, id: category_2.id }
        end.not_to change { category_2 }
        expect(response).to render_template(:edit)
      end
    end
  end

  describe 'DELETE destroy' do
    specify do
      expect do
        delete :destroy, params: { id: user_1.id }
      end.to change(User, :count).by(-1)
      expect(response).to redirect_to(users_path)
    end
  end
end
```