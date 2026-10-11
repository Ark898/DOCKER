# Use a specific LTS Node.js version instead of the floating `node` tag,
# which can pull a vulnerable base image and increase the package count.
FROM node:20.12.2-alpine3.20

# Particular version of Node.js
# From node:14

# Set the working directory inside the container
WORKDIR /testapp

# Copy the content of the current directory to the working directory inside the container
COPY . .

# Install dependencies
RUN npm install

# Expose the port that the application will run on
EXPOSE 5173

# Start the application
CMD ["npm", "run", "dev"]




